| | issue |
| ----------- | ----------- |
| 1 | [State variables only set in the constructor should be declared immutable.](#1-state-variables-only-set-in-the-constructor-should-be-declared-immutable-ones-not-caught-by-bot) |
| 2 | [state variables can be packed into fewer storage slots](#2-state-variables-can-be-packed-into-fewer-storage-slots) |
| 3 | [state variables should be cached in stack variables rather than re-reading them from storage](#3-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-ones-not-caught-by-bot) |
| 4 | [not using the named return variables when a function returns, wastes deployment gas](#4-not-using-the-named-return-variables-when-a-function-returns-wastes-deployment-gas) |
| 5 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#5-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied) |
| 6 | [require()/revert() strings longer than 32 bytes cost extra gas](#6-requirerevert-strings-longer-than-32-bytes-cost-extra-gas) |
| 7 | [require() or revert() statements that check input arguments should be at the top of the function](#7-require-or-revert-statements-that-check-input-arguments-should-be-at-the-top-of-the-function) |
| 8 | [use custom errors rather than revert()/require() strings to save deployment gas](#8-use-custom-errors-rather-than-revertrequire-strings-to-save-deployment-gas) |
| 9 | [using private rather than public for constants, saves gas](#9-using-private-rather-than-public-for-constants-saves-gas) |
| 10 | [state var is defined but not used anywhere but](#10-state-var-is-defined-but-not-used-anywhere) |
| 11 | [Functions guaranteed to revert when called by normal users can be marked `payable`](#11-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable) |
| 12 | [Optimize names to save gas](#12-optimize-names-to-save-gas) |
| 13 | [sorting the lines better will result in less storage access needed which result in gas save](#13-sorting-the-lines-better-will-result-in-less-storage-access-needed-which-result-in-gas-save) |
| 14 | [Use hardcoded address instead address(this)](#14-use-hardcoded-address-instead-addressthis) |
| 15 | [Unbounded Gas Consumption Risk Due to External Call Recipients](#15-unbounded-gas-consumption-risk-due-to-external-call-recipients) |
| 16 | [Refactor `If/require` statements to save gas in case of early revert](#16-refactor-ifrequire-statements-to-save-gas-in-case-of-early-revert) |

## 1. State variables only set in the constructor should be declared immutable. (ones not caught by bot)

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmaccess (100 gas) with a PUSH32 (3 gas). While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

`token0`
- [GeVault.sol#L48](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L48)

`token1`
- [GeVault.sol#L49](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L49)

`ROEROUTER`
- [PositionManager.sol#L23](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L23)


although for these ones they are turned into state variables to fix for coverage testing, make sure to turn them into immutable after the bug is checked or resolved
- [GeVault.sol#L59](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L59)
- [GeVault.sol#L61](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L61)


## 2. state variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

bring `isEnabled`(#L50) and `baseTokenIsToken0`(#L64) which are booleans together with `treasury`(#L58) to put them into one slot instead of current 3. both bools have 1 access with `treasury` resulting in some gas save for accessing them.
- [GeVault.sol#L58](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L58)

bring `liquidity`(#L52) to `lowerTick`(#L28). 1 less slot used and also there is accesses to `liquidity` and `lowerTick` or `upperTick` or `feeTier` in 4 places, so the accesses gonna be cheaper.
- [TokenisableRange.sol#L52](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L52)


## 3. state variables should be cached in stack variables rather than re-reading them from storage (ones not caught by bot)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

`ticks.length` can be cached before #L151 because its at least read twice from storage (#L151 and #L153) and if `ticks.length > 2` coniditon be true, its gonna be read from storage 3 times every iteration of the loop. potentially saves a lot of gas. (make sure to adjust the variable we make after the push #L151)
- [GeVault.sol#L154](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L154)

`token0` can be cached before #L251 to save 97 gas. its read from storage in both #L251 and #L266.(a better version of this is to cache all of this `token == address(token0)` to have one less comparison and cast but it reduces readability)
- [GeVault.sol#L266](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L266)

`baseFeeX4` will be read more than twice without changing value. cache it before #L451. saves at least 197 gas or possibly 397 gas.
- [GeVault.sol#L451](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L451)

`address(tokenisedRanges[step])` can be cached before #L114 to reduce 2 complex SLOADs. used in #L114 and #L118 and #L120
- [RangeManager.sol#L114](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L114)

`address(tokenisedTicker[step])` can be cached before #L126 to reduce 2 complex SLOADs. used in #L126 and #L130 and #L132
- [RangeManager.sol#L126](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L126)

`address(ASSET_0)` can be cached before #L154 to reduce on SLOAD saving 97 gas. used in #L154 and #L155
- [RangeManager.sol#L155](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L155)

`address(ASSET_1)` can be cached before #L159 to reduce on SLOAD saving 97 gas. used in #L159 and #L160
- [RangeManager.sol#L160](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L160)

`fee0` can be cached before #L241 to at least save 97 gas(if the condition in #L241 is true so we save 297 gas). for this we need to change the #L243 from += into equals and use the cached version instead of state var.
- [TokenisableRange.sol#L241](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L241)

`fee1` can be cached before #L242 to at least save 97 gas(if the condition in #L242 is true so we save 297 gas). for this we need to change the #L244 from += into equals and use the cached version instead of state var.
- [TokenisableRange.sol#L242](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L242)

`TOKEN0.decimals` can be cached before #L274 to save gas. its used in #L274 and #L275. (a better version can be caching all this `TOKEN0_PRICE / 10 ** TOKEN0.decimals` to not do calculations again)
- [TokenisableRange.sol#L274](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L274)

`TOKEN1.decimals` can be cached before #L274 to save gas. its used in #L274 and #L275. (a better version can be caching all this `TOKEN1_PRICE / 10 ** TOKEN1.decimals` to not do calculations again)
- [TokenisableRange.sol#L274](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L274)

`liquidity` can be cached before #L237 to at least save 97 gas (possibly 197 if the condition in #L237 be true). used in #L278 and #L279 and a possible #L240.
- [TokenisableRange.sol#L278](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L278)


## 4. not using the named return variables when a function returns, wastes deployment gas

- [TokenisableRange.sol#L352](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352)
- [TokenisableRange.sol#L361](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L361)

- [OracleConvert.sol#L78](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/#L78)


## 5. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

- [GeVault.sol#L154](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L154)
- [GeVault.sol#L299](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L299)
- [GeVault.sol#L314](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L314)
- [GeVault.sol#L431](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L431)

- [RangeManager.sol#L62](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L62)

- [OptionsPositionManager.sol#L71](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L71)
- [OptionsPositionManager.sol#L172](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L172)
- [OptionsPositionManager.sol#L203](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L203)


## 6. require()/revert() strings longer than 32 bytes cost extra gas

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas

- [FixedOracle.sol#L11](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11)


## 7. require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

the require in #L218 is cheaper than all above it so we can put it first for possible gas save
- [GeVault.sol#L218](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L218)

the require in #L252 is cheaper than all above it so we can put it first for possible gas save
- [GeVault.sol#L252](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L252)


## 8. use custom errors rather than revert()/require() strings to save deployment gas

Custom errors are available from solidity version 0.8.4. Custom errors save ~50 gas each time they’re hit by avoiding having to allocate and store the revert string. Not defining the strings also save deployment gas

https://blog.soliditylang.org/2021/04/21/custom-errors/


- [GeVault.sol#L79](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L79)
- [GeVault.sol#L80](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L80)
- [GeVault.sol#L81](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L81)
- [GeVault.sol#L120](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120)
- [GeVault.sol#L125](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L125)
- [GeVault.sol#L127](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L127)
- [GeVault.sol#L141](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L141)
- [GeVault.sol#L146](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L146)
- [GeVault.sol#L148](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L148)
- [GeVault.sol#L170](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L170)
- [GeVault.sol#L184](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L184)
- [GeVault.sol#L203](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L203)
- [GeVault.sol#L215](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L215)
- [GeVault.sol#L217](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L217)
- [GeVault.sol#L218](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L218)
- [GeVault.sol#L249](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L249)
- [GeVault.sol#L250](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L250)
- [GeVault.sol#L251](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L251)
- [GeVault.sol#L252](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L252)
- [GeVault.sol#L256](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L256)
- [GeVault.sol#L269](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L269)
- [GeVault.sol#L281](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L281)

- [RangeManager.sol#L49](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L49)
- [RangeManager.sol#L60](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L60)
- [RangeManager.sol#L76](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L76)
- [RangeManager.sol#L108](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L108)
- [RangeManager.sol#L206](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L206)

- [RoeRouter.sol#L34](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L34)
- [RoeRouter.sol#L68](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L68)
- [RoeRouter.sol#L75](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L75)
- [RoeRouter.sol#L84](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L84)

- [TokenisableRange.sol#L86](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L86)
- [TokenisableRange.sol#L87](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L87)
- [TokenisableRange.sol#L135](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L135)
- [TokenisableRange.sol#L136](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L136)
- [TokenisableRange.sol#L209](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L209)
- [TokenisableRange.sol#L225](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L225)
- [TokenisableRange.sol#L271](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L271)

- [OptionsPositionManager.sol#L69](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L69)
- [OptionsPositionManager.sol#L91](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L91)
- [OptionsPositionManager.sol#L137](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L137)
- [OptionsPositionManager.sol#L167](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L167)
- [OptionsPositionManager.sol#L198](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L198)
- [OptionsPositionManager.sol#L235](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L235)
- [OptionsPositionManager.sol#L261](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L261)
- [OptionsPositionManager.sol#L344](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L344)
- [OptionsPositionManager.sol#L369](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L369)
- [OptionsPositionManager.sol#L393](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L393)
- [OptionsPositionManager.sol#L420](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L420)
- [OptionsPositionManager.sol#L441](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L441)
- [OptionsPositionManager.sol#L495](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L495)
- [OptionsPositionManager.sol#L496](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L496)
- [OptionsPositionManager.sol#L523](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L523)
- [OptionsPositionManager.sol#L525](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L525)
- [OptionsPositionManager.sol#L536](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L536)

- [PositionManager.sol#L30](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L30)
- [PositionManager.sol#L145](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L145)
- [PositionManager.sol#L146](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L146)

- [FixedOracle.sol#L11](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11)

- [LPOracle.sol#L29](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L29)
- [LPOracle.sol#L63](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L63)
- [LPOracle.sol#L101](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L101)

- [OracleConvert.sol#L23](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/#L23)
- [OracleConvert.sol#L26](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/#L26)
- [OracleConvert.sol#L44](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/#L44)

- [V3Proxy.sol#L87](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L87)
- [V3Proxy.sol#L91](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L91)
- [V3Proxy.sol#L99](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L99)
- [V3Proxy.sol#L106](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L106)
- [V3Proxy.sol#L113](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L113)
- [V3Proxy.sol#L125](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L125)
- [V3Proxy.sol#L138](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L138)
- [V3Proxy.sol#L139](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L139)
- [V3Proxy.sol#L148](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L148)
- [V3Proxy.sol#L149](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L149)
- [V3Proxy.sol#L161](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L161)
- [V3Proxy.sol#L162](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L162)
- [V3Proxy.sol#L179](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L179)
- [V3Proxy.sol#L180](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L180)


## 9. using private rather than public for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table

- [GeVault.sol#L62](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L62)

- [RangeManager.sol#L36](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36)

- [TokenisableRange.sol#L58](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L58)
- [TokenisableRange.sol#L59](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L59)
- [TokenisableRange.sol#L60](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L60)
- [TokenisableRange.sol#L61](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L61)


## 10. state var is defined but not used anywhere

even if they are for readability, consider making them comments instead. saves a lot of deplyment gas

- [GeVault.sol#L41](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L41)


## 11. Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

- [GeVault.sol#L101](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101)
- [GeVault.sol#L108](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L108)
- [GeVault.sol#L116](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L116)
- [GeVault.sol#L137](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L137)
- [GeVault.sol#L167](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L167)
- [GeVault.sol#L183](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L183)
- [GeVault.sol#L191](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L191)

- [RangeManager.sol#L75](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75)
- [RangeManager.sol#L95](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L95)

- [RoeRouter.sol#L48](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L48)
- [RoeRouter.sol#L65](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L65)
- [RoeRouter.sol#L83](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L83)

- [FixedOracle.sol#L24](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24)

- [V3Proxy.sol#L94](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L94)


## 12. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signitures 


## 13. sorting the lines better will result in less storage access needed which result in gas save

after we sort these lines like this, less gas will be used for storage access because some of these variables are inside one slot of storage

after the changes mentioned before [here](#2-state-variables-can-be-packed-into-fewer-storage-slots):
bring #L89 near #L92 because both `treasury` and `baseTokenIsToken0` are inside one slot so it takes less gas to assign to them
- [GeVault.sol#L89](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L89)


## 14. Use hardcoded address instead address(this)

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this. - [Reference](https://twitter.com/transmissions11/status/1518507047943245824)

- [GeVault.sol#L262](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L262)
- [GeVault.sol#L302](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L302)
- [GeVault.sol#L324](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L324)
- [GeVault.sol#L330](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L330)
- [GeVault.sol#L339](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L339)
- [GeVault.sol#L340](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L340)
- [GeVault.sol#L386](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L386)
- [GeVault.sol#L409](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L409)
- [GeVault.sol#L412](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L412)
- [GeVault.sol#L423](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L423)

- [RangeManager.sol#L96](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L96)
- [RangeManager.sol#L98](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L98)
- [RangeManager.sol#L101](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L101)
- [RangeManager.sol#L118](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L118)
- [RangeManager.sol#L130](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L130)
- [RangeManager.sol#L155](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L155)
- [RangeManager.sol#L160](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L160)
- [RangeManager.sol#L191](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L191)
- [RangeManager.sol#L192](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L192)

- [TokenisableRange.sol#L138](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138)
- [TokenisableRange.sol#L139](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L139)
- [TokenisableRange.sol#L153](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L153)
- [TokenisableRange.sol#L159](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L159)
- [TokenisableRange.sol#L160](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L160)
- [TokenisableRange.sol#L171](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L171)
- [TokenisableRange.sol#L228](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L228)
- [TokenisableRange.sol#L229](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L229)

- [OptionsPositionManager.sol#L92](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L92)
- [OptionsPositionManager.sol#L103](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L103)
- [OptionsPositionManager.sol#L104](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L104)
- [OptionsPositionManager.sol#L105](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L105)
- [OptionsPositionManager.sol#L176](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L176)
- [OptionsPositionManager.sol#L207](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L207)
- [OptionsPositionManager.sol#L275](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L275)
- [OptionsPositionManager.sol#L310](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L310)
- [OptionsPositionManager.sol#L312](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L312)
- [OptionsPositionManager.sol#L313](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L313)
- [OptionsPositionManager.sol#L369](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L369)
- [OptionsPositionManager.sol#L479](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L479)
- [OptionsPositionManager.sol#L496](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L496)
- [OptionsPositionManager.sol#L501](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L501)

- [PositionManager.sol#L81](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L81)
- [PositionManager.sol#L90](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L90)
- [PositionManager.sol#L115](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L115)
- [PositionManager.sol#L127](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L127)

- [V3Proxy.sol#L95](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L95)
- [V3Proxy.sol#L115](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L115)
- [V3Proxy.sol#L127](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L127)
- [V3Proxy.sol#L132](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L132)
- [V3Proxy.sol#L164](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L164)
- [V3Proxy.sol#L167](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L167)
- [V3Proxy.sol#L182](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L182)
- [V3Proxy.sol#L186](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L186)


## 15. Unbounded Gas Consumption Risk Due to External Call Recipients

In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using `addr.call`{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

- [V3Proxy.sol#L156](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156)
- [V3Proxy.sol#L174](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L174)
- [V3Proxy.sol#L192](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L192)


## 16. Refactor `If/require` statements to save gas in case of early revert

Checks that involve calldata should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before using excessive gas in a call that may ultimately revert in an unhappy case.

first part of the `if` has 2 SLOADs and because there is && in between the if conditions putting the first part as last will have a chance of saving gas by doing one less SLOAD. put `fee0+fee1 > 0 ` check last one to be checked
- [TokenisableRange.sol#L237](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L237)
