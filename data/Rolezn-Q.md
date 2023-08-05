## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW-QA&#x2011;3) | Array lengths not checked | 2 |
| [LOW&#x2011;2](#LOW-QA&#x2011;5) | Consider the case where `totalsupply` is 0 | 14 |
| [LOW&#x2011;3](#LOW-QA&#x2011;6) | Constant decimal values | 7 |
| [LOW&#x2011;4](#LOW-QA&#x2011;7) | Do not allow fees to be set to `100%` | 1 |
| [LOW&#x2011;5](#LOW-QA&#x2011;14) | Function could be used to add with an arbitary id, which could be problematic in upcoming future | 1 |
| [LOW&#x2011;6](#LOW-QA&#x2011;16) | Large transfers may not work with some `ERC20` tokens | 3 |
| [LOW&#x2011;7](#LOW-QA&#x2011;25) | Remove unused code | 1 |
| [LOW&#x2011;8](#LOW-QA&#x2011;31) | Usage of `payable.transfer` can lead to loss of funds | 1 |

Total: 57 contexts over 8 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC-QA&#x2011;1) | `address` shouldn't be hard-coded | 2 |
| [NC&#x2011;2](#NC-QA&#x2011;13) | Declare interfaces on separate files | 6 |
| [NC&#x2011;3](#NC-QA&#x2011;14) | Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function | 24 |
| [NC&#x2011;4](#NC-QA&#x2011;15) | Duplicate imports | 4 |
| [NC&#x2011;5](#NC-QA&#x2011;23) | Some functions contain the same exact logic | 4 |
| [NC&#x2011;6](#NC-QA&#x2011;45) | Omissions in Events | 1 |
| [NC&#x2011;7](#NC-QA&#x2011;47) | `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings | 2 |
| [NC&#x2011;8](#NC-QA&#x2011;62) | Zero as a function argument should have a descriptive meaning | 20 |

Total: 63 contexts over 8 issues

## Low Risk Issues

### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Array lengths not checked

If the length of the arrays are not required to be of the same length, user operations may not be fully executed

#### <ins>Proof Of Concept</ins>

```solidity
File: OptionsPositionManager.sol


function executeBuyOptions(
    uint poolId,
    address[] calldata assets,
    uint256[] calldata amounts,
    address user,
    address[] memory sourceSwap
  ) internal {

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L61

```solidity
File: OptionsPositionManager.sol


function executeLiquidation(
    uint poolId,
    address[] calldata assets,
    uint256[] calldata amounts,
    address user,
    address collateral
  ) internal {

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L83



### <a href="#qa-summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Consider the case where `totalsupply` is 0

Consider the case where `totalsupply` is 0. When `totalsupply` is 0, it should return 0 directly, because there will be an error of dividing by 0.

#### <ins>Impact</ins>

This would cause the affected functions to revert and as a result can lead to potential loss.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

221: uint valueX8 = vaultValueX8 * liquidity / totalSupply();

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L221

```solidity
File: GeVault.sol

293: priceX8 = vaultValue * 1e18 / supply;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L293

```solidity
File: TokenisableRange.sol

294: uint removedLiquidity = uint(liquidity) * lp / totalSupply();
316: TOKEN0.token.safeTransfer( msg.sender, fee0 * lp / totalSupply());
317: removed0 += fee0 * lp / totalSupply();
318: fee0 -= fee0 * lp / totalSupply();
321: TOKEN1.token.safeTransfer(  msg.sender, fee1 * lp / totalSupply());
322: removed1 += fee1 * lp / totalSupply();
323: fee1 -= fee1 * lp / totalSupply();

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L294

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L316

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L317

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L318

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L321

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L322

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L323



```solidity
File: TokenisableRange.sol

356: return totalValue * 1e18 / totalSupply();

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L356

```solidity
File: TokenisableRange.sol

371: (token0Amount, token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  uint128 ( uint(liquidity) * amount / totalSupply() ) );

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L371

```solidity
File: TokenisableRange.sol

379: token0Amount += fee0 * amount / totalSupply();
380: token1Amount += fee1 * amount / totalSupply();

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L379-L380

```solidity
File: LPOracle.sol

103: return int(val / LP_TOKEN.totalSupply());

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/LPOracle.sol#L103



</details>


#### <ins>Recommended Mitigation Steps</ins>
Add check for zero value and return 0.

```solidity
if ( totalSupply() == 0) return 0;
```



### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Constant decimal values

The use of fixed decimal values such as 1e18 or 1e8 in Solidity contracts can lead to inaccuracies, bugs, and vulnerabilities, particularly when interacting with tokens having different decimal configurations. Not all ERC20 tokens follow the standard 18 decimal places, and assumptions about decimal places can lead to miscalculations.

Resolution: Always retrieve and use the `decimals()` function from the token contract itself when performing calculations involving token amounts. This ensures that your contract correctly handles tokens with any number of decimal places, mitigating the risk of numerical errors or under/overflows that could jeopardize contract integrity and user funds.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

275: liquidity = valueX8 * 1e10;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L275

```solidity
File: GeVault.sol

293: priceX8 = vaultValue * 1e18 / supply;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L293

```solidity
File: GeVault.sol

396: valueX8 += bal * t.latestAnswer() / 1e18;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L396

```solidity
File: TokenisableRange.sol

99: int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );
100: int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L99-L100

```solidity
File: TokenisableRange.sol

356: return totalValue * 1e18 / totalSupply();

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L356

```solidity
File: OptionsPositionManager.sol

341: uint debtValue = TokenisableRange(debtAsset).latestAnswer() * debtAmount / 1e18;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L341



</details>




### <a href="#qa-summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Do not allow fees to be set to `100%`

It is recommended from a risk perspective to disallow setting 100% fees at all. A reasonable fee maximum should be checked for instead.
There is currently no limit to what `newBaseFeeX4` can be set to.

#### <ins>Proof Of Concept</ins>


```solidity
File: GeVault.sol


function setBaseFee(uint newBaseFeeX4) public onlyOwner {
  require(newBaseFeeX4 < 1e4, "GEV: Invalid Base Fee");
    baseFeeX4 = newBaseFeeX4;
    emit SetFee(newBaseFeeX4);
  }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L183


### <a href="#qa-summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Large transfers may not work with some `ERC20` tokens

Some `IERC20` implementations (e.g `UNI`, `COMP`) may fail if the valued transferred is larger than `uint96`. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

227: ERC20(token).safeTransfer(treasury, fee);
267: ERC20(token).safeTransfer(treasury, fee);
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L227

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L267


```solidity
File: OptionsPositionManager.sol

370: IERC20(collateralAsset).safeTransfer(ROEROUTER.treasury(), feeAmount);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L370


</details>


### <a href="#qa-summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Remove unused code
This code is not used in the main project contract files, remove it or add event-emit
Code that is not in use, suggests that they should not be present and could potentially contain insecure functionalities.

#### <ins>Proof Of Concept</ins>


```solidity
File: PositionManager.sol

function validateValuesAgainstOracle
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/PositionManager.sol#L138






### <a href="#qa-summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> Usage of `payable.transfer` can lead to loss of funds 
The funds that are to be sent can be lost. The issues with `transfer()` and `send()` are outlined <a href="https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/">here</a>:
The `transfer()` call requires that the recipient to be either an EOA account, or a contract that has a `payable` callback. For the contract case, the `transfer()` call only provides 2300 gas for the contract to complete its operations. This means the following cases can cause the transfer to fail:

- The contract does not have a `payable` callback
- The contract's `payable` callback spends more than 2300 gas (which is only enough to emit something)
- The contract is called through a proxy which itself uses up the 2300 gas Use OpenZeppelin's `Address.sendValue()` instead

#### <ins>Proof Of Concept</ins>


```solidity
File: GeVault.sol

232: payable(msg.sender).transfer(bal);
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L232



#### <ins>Recommended Mitigation Steps</ins>

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60


## Non Critical Issues

### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> `address` shouldn't be hard-coded

It is often better to declare `address`es as `immutable`, and assign them via constructor arguments. This allows the code to remain the same across deployments on different networks, and avoids recompilation when addresses need to change.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: TokenisableRange.sol

54: address public TREASURY_DEPRECATED = 0x22Cc3f665ba4C898226353B672c5123c58751692;
60: address constant public treasury = 0x22Cc3f665ba4C898226353B672c5123c58751692;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L54

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L60

</details>


### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Declare interfaces on separate files

Interfaces should be declared on a separate file and not on the same file where the contract resides in.

#### <ins>Proof Of Concept</ins>

```solidity
File: LPOracle.sol

6: interface UniswapV2Pair {
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/LPOracle.sol#L6

```solidity
File: LPOracle.sol

13: interface IERC20 {
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/LPOracle.sol#L13

```solidity
File: V3Proxy.sol

9: interface ISwapRouter {
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L9

```solidity
File: V3Proxy.sol

38: interface IQuoter {
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L38

```solidity
File: V3Proxy.sol

56: interface IWETH9 is IERC20 {
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L56

```solidity
File: OptionsPositionManager.sol

7: interface  AmountsRouter {
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L7






#### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs

#### <ins>Proof Of Concept</ins>

```solidity
File: GeVault.sol

120: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
141: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
170: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
203: require(poolMatchesOracle(), "GEV: Oracle Error");
215: require(poolMatchesOracle(), "GEV: Oracle Error");
250: require(poolMatchesOracle(), "GEV: Oracle Error");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L120

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L141

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L170

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L203

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L215

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L250



```solidity
File: V3Proxy.sol

87: require(acceptPayable, "CannotReceiveETH");
91: require(acceptPayable, "CannotReceiveETH");
99: require(path.length == 2, "Direct swap only");
106: require(path.length == 2, "Direct swap only");
113: require(path.length == 2, "Direct swap only");
125: require(path.length == 2, "Direct swap only");
138: require(path.length == 2, "Direct swap only");
148: require(path.length == 2, "Direct swap only");
161: require(path.length == 2, "Direct swap only");
179: require(path.length == 2, "Direct swap only");
139: require(path[0] == ROUTER.WETH9(), "Invalid path");
149: require(path[0] == ROUTER.WETH9(), "Invalid path");
162: require(path[1] == ROUTER.WETH9(), "Invalid path");
180: require(path[1] == ROUTER.WETH9(), "Invalid path");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L87

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L91

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L99

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L106

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L113

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L125

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L138

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L148

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L161

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L179

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L139

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L149

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L162

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L180



```solidity
File: OptionsPositionManager.sol

69: require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");
91: require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");
393: require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );
420: require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L69

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L91

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L393

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L420








### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Duplicate imports

There are several occasions where there several imports of the same file

#### <ins>Proof Of Concept</ins>


```solidity
File: RangeManager.sol

5: import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";
7: import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L5

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L7



```solidity
File: TokenisableRange.sol

13: import "../interfaces/IAaveOracle.sol";
14: import "../interfaces/IAaveOracle.sol";

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L13-L14



### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Some functions contain the same exact logic

These functions might be a problem if the logic changes before the contract is deployed, as the developer must remember to syncronize the logic between all the function instances.

Consider using a single function instead of duplicating the code, for example by using a `library`, or through inheritance.

#### <ins>Proof Of Concept</ins>


```solidity
File: LPOracle.sol

38: function decimals() external pure returns (uint8) {
    return 8;
  }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/LPOracle.sol#L38

```solidity
File: OracleConvert.sol

30: function decimals() external pure returns (uint8) {
    return 8;
  }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/OracleConvert.sol#L30

```solidity
File: OracleConvert.sol

37: function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {
    (
      , 
      int price,
      ,
      uint timeStamp,
    ) = priceFeed.latestRoundData();
    require(timeStamp > 0, "Round not complete");
    return price;
  }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/OracleConvert.sol#L37

```solidity
File: LPOracle.sol

56: function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {
    (
      , 
      int price,
      ,
      uint timeStamp,
    ) = priceFeed.latestRoundData();
    require(timeStamp > 0, "Round not complete");
    return price;
  }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/LPOracle.sol#L56



### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters
The following events should also add the previous original value in addition to the new value.

#### <ins>Proof Of Concept</ins>


```solidity
File: RoeRouter.sol

14: event UpdateTreasury(address treasury);
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RoeRouter.sol#L14




#### <ins>Recommended Mitigation Steps</ins>
The events should include the new value and old value where possible.




### <a href="#qa-summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> `override` function arguments that are unused should have the variable name removed or commented out to avoid compiler warnings

#### <ins>Proof Of Concept</ins>


```solidity
File: OptionsPositionManager.sol

38: function executeOperation : uint256[] calldata premiums
38: function executeOperation : address initiator

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L38



### <a href="#qa-summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Zero as a function argument should have a descriptive meaning

Consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: GeVault.sol

357: depositAndStash(ticks[tick1Index], 0, availToken1 / 2);
358: depositAndStash(ticks[tick1Index+1], 0, availToken1 / 2);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L357-L358

```solidity
File: RangeManager.sol

119: tokenisedRanges[step].withdraw(trAmt, 0, 0);
131: tokenisedTicker[step].withdraw(ttAmt, 0, 0);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L119

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L131



```solidity
File: TokenisableRange.sol

198: amount0Min: 0,
199: amount1Min: 0,

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L198-L199

```solidity
File: OptionsPositionManager.sol

135: (uint256 amount0, uint256 amount1) = TokenisableRange(flashAsset).withdraw(flashAmount, 0, 0);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L135

</details>