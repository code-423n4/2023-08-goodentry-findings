# No function to remove a tick
## Summary

In the `GeVault.sol` it is impossible to remove a `tick` once it is added to the `ticks[]`.

## Vulnerability Details

There are methods to add, shift and modify ticks but none to remove ticks from the ticks array.

But if due to a human error a faulty tick is added, there won't be any way to remove it. It could only be overridden to 0 and moved left to avoid breaking other functions, a non-ideal solution.

Therefore, it is advisable to implement another function to allow removing a tick.

## Impact

It won't allow removing faulty added ticks, being the only solution to override them to 0 and move them to the left, to avoid breaking other functions.

## Proof of Concept

- `pushTick` function: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/GeVault.sol#L116
- `shiftTick` function: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/GeVault.sol#L137
- `modifyTick` function: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/GeVault.sol#L167

## Tools Used

Manual review.

## Recommended Mitigation Steps
Add a function to remove a faulty tick from the ticks array.

---

# Transactions calling addDust revert if token has more than 20 decimals
## Summary

In the `addDust` function in the `OptionsPositionManager.sol` file, if `token0` or `token1` has more than 20 decimals, the transaction will always revert.

## Vulnerability Details

The function executes some code to scale the tokens value:
```solidity
uint scale0 = 10**(20 - ERC20(token0).decimals()) * oracle.getAssetPrice(token0) / 1e8;
uint scale1 = 10**(20 - ERC20(token1).decimals()) * oracle.getAssetPrice(token1) / 1e8;
```

But since they are doing `20 - token0Decimals` and `20 - token1Decimals`, if one of those tokens has more than 20 decimals, the subtraction will underflow causing the entire transaction to revert, making it impossible to run this function for that token pair.

## Impact

The token pairs having at least one token with more than 20 decimals won't be able to be used in the `addDust` execution since those parameters will always cause the function to revert.

## Proof of Concept

- Affected lines: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/PositionManager/OptionsPositionManager.sol#L544-L551

## Tools Used

Manual review.

## Recommended Mitigation Steps

Since it is already doing some logic to support different token decimals, a conditional to check if the token decimals are higher than 20 could be implemented, and then some logic to support tokens with more than 20 decimals.

---

# Math always revert for tokens with 39 or more decimals
## Summary

In the `initProxy` function of the `TokenisableRange.sol` contract, if `TOKEN0` or `TOKEN1` has 39 or more decimals, the math will overflow, causing the entire function to revert.

## Vulnerability Details

In the `initProxy` function there is some math to calculate the upper and lower ticks used (https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/TokenisableRange.sol#L99-L100):
```solidity
 int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );
    int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );
```

But if `TOKEN0.decimals` or `TOKEN1.decimals` have 39 or more decimals, the math will overflow causing the entire transaction to revert, making it impossible to initialize the contract for that token pair.

## Impact

This makes it impossible to use this protocol feature with pairs that have at least one token with 39 or more decimals. Not being able to use some protocol features is a high-severity issue but since there are not many tokens using 39 or more decimals we set the impact to low.

## Proof of Concept

- Lines where the issue happens: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/TokenisableRange.sol#L99-L100

Just try running in Solidity the following math with the following params:
- `TOKEN1.decimals = 39`
- `TOKEN0.decimals = 18`
- `startX10 = 1e10` - Comments in the code explain that this is a value scaled by 1e10, so we can use 1e10 as a demo value

You can try the following contract:
```solidity
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.8.2 <0.9.0;

contract TestContract {
    function sqrt(uint x) internal pure returns (uint y) {
      uint z = (x + 1) / 2;
      y = x;
      while (z < y) {
          y = z;
          z = (x / z + z) / 2;
      }
    }

    function testThatReverts() public pure {
        uint256 TOKEN1Decimals = 39;
        uint256 TOKEN0Decimals = 18;
        uint256 startX10 = 1e10;
        uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1Decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0Decimals) ) );
    }

    function testThatWorks() public pure {
        uint256 TOKEN1Decimals = 38;
        uint256 TOKEN0Decimals = 18;
        uint256 startX10 = 1e10;
        uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1Decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0Decimals) ) );
    }
}
```

## Tools Used

Manual review, Foundry fuzzing, and Remix.

## Recommended Mitigation Steps

Restrict the decimals supported, change the way the math is done, or use a library like `PBRMath` to support bigger numbers without causing overflow (https://github.com/PaulRBerg/prb-math).

---

# Sqrt function does not work in every case
## Summary

The `sqrt` function used in the project fails due to overflow in some cases, making it suboptimal to use.

## Vulnerability Details

The `sqrt` function defined [here](https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/TokenisableRange.sol#L66) and also [here](https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/lib/Sqrt.sol#L8) (which by the way it should always use the second, instead of having duplicated code in other contracts) does not work in every number in the uint256 range.

After doing some fuzzy testing we found it fails when doing the square root of the max uint256 number (`2**256 -1`). That overflow happens because inside the `sqrt` function, they sum 1 to the entered argument value and since we entered max uint256, you can not add 1 to that value without causing an overflow.

It is clear this is an edge case but a proper `sqrt` function defined to accept a `uint256` input should support every possible value inside the supported input type. In this case, `sqrt` should support any value from `0` to `2**256-1`, which is not the case.

## Impact

Suboptimal input type support can cause unexpected reverts when trying to do the `sqrt` of some inputs.

## Proof of Concept

- Case 1: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/TokenisableRange.sol#L66
- Case 2: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/lib/Sqrt.sol#L8
- Cause of the overflow with an input value of `2**256-1`: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/lib/Sqrt.sol#L9

## Tools Used

Manual review and fuzzy testing with Foundry.

## Recommended Mitigation Steps

Our recommendation is to use another `sqrt` function. There are two battle-tested options that are the market standards:
- One would be the one used by Uniswap V2, which supports `2**256-1` without reverting: https://ethereum.stackexchange.com/a/87713
- The other one would be using the popular `PBRMath` library, which has a `sqrt` method that also supports every input value: https://github.com/PaulRBerg/prb-math

---

# Missing check to make sure added tokens corresponds to the oracle/pool
## Summary

In the `LPOracle.sol` and `RoeRouter.sol` files, there are functions that don't verify if the tokens sent in the params are part of the oracle/pool address also sent in the params. That can allow human mistakes that lead to wrong price calculations. 

## Vulnerability Details

There is no check to make sure that the tokens constituting the LP/oracle are the same as the chainlink price tokens. Due to human error, a mistake can be made and you end up with a price for token 0 with decimals of token 1, which would completely destroy any price calculations.

## Impact

Potential wrong price calculations if the wrong tokens or the right tokens but in a wrong order are submitted when deploying the oracle.

## Proof of Concept

- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/helper/LPOracle.sol#L28-L29
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/RoeRouter.sol#L54-L79

## Tools Used

Manual review.

## Recommended Mitigation Steps

Implement a check to verify the tokens added in the params are the same (and in the same order) as the one used in the LP/oracle added also in the params.

---

# Math not following natspec
## Summary

Unexpected return range based on the specified natspec docs.

## Vulnerability Details

In this case, natspec function comment says its returned value follows a linear model: from `baseFeeX4 / 2` to `baseFeeX4 * 2`.

But in the code, if `adjustedBaseFeeX4` is greater than `baseFeeX4 * 3/2` (which is smaller than `baseFeeX4 * 2`), the code will set the `adjustedBaseFeeX4` to `baseFeeX4 * 3/2`; making impossible to reach a value of `baseFeeX4 * 2` ever although being stated like that in the natspec.

Above that function there is a comment saying `// Adjust from -50% to +50%` but again, that doesn't follow the defined natspec of the function, which are the ruling comments in a function.

## Impact

Since NatSpec documentation is considered to be part of the contract’s public API, NatSpec-related issues are assigned a higher severity than other code-comment-related issues and the code **must** follow the specified natspec, which is not happening.

## Proof of Concept

- Return range is defined in the following Natspec: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/GeVault.sol#L443
- But wrong max range is set here: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/GeVault.sol#L457

## Tools Used

Manual review.

## Recommended Mitigation Steps

There are two options:
1. Change the Natspec like this:
```diff
- /// @dev Simple linear model: from baseFeeX4 / 2 to baseFeeX4 * 2
+ /// @dev Simple linear model: from baseFeeX4 / 2 to baseFeeX4 * 3 /2
```

2. Change the max range math to this:
```diff
- if (adjustedBaseFeeX4 > baseFeeX4 * 3 / 2) adjustedBaseFeeX4 = baseFeeX4 * 3 / 2;
+ if (adjustedBaseFeeX4 > baseFeeX4 * 2) adjustedBaseFeeX4 = baseFeeX4 * 2;
```

---

# Very old OpenZeppelin version being used
## Summary

A very old version of OpenZeppelin is being used (version `4.4.1`).

## Vulnerability Details

As said in the summary, the project is using a really old OpenZeppelin version (`4.4.1`), the latest stable one the `4.9.3`.

Using old versions can lead to less gas performance and security issues. We checked for known issues and although there are some known issues in the version being used, none of the audited contracts are using those features. But in any case, it is not recommended to use an old version so our recommendation is to upgrade to the latest stable version.

## Impact

Using an old version can lead to worse gas performance, known contract issues, and potential 0-day issues.

## Proof of Concept

- Known issues of the used old version: https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/4.4.1

## Tools Used

Manual review and Snyk.io.

## Recommended Mitigation Steps

Upgrade to the latest stable OpenZeppelin version. At the time of the audit, the latest stable version is the `4.9.3`.

---

# Dead code due in some contracts
## Summary

There are some occurrences of unused (`dead`) code in some contracts.

## Vulnerability Details

Avoiding to have unused code in the contracts is a good practice since it helps to keep the code clean and readable.

In `RangeManager.sol`, the `POS_MGR` constant is declared but not being used, so it could be removed: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/RangeManager.sol#L36C47-L36C55

In `GeVault.sol`, the `rangeManager` is being declared but it is not being used so it could also be removed. Also, if this variable is removed, the import `import "./RangeManager.sol";` could also be deleted.

## Impact

As said before, removing unused code can keep the code clean and more readable.

## Proof of Concept

Affected lines:
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/RangeManager.sol#L36C47-L36C55
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/GeVault.sol#L41

## Tools Used

Manual review.

## Recommended Mitigation Steps

Remove the unused code.

---

# Inconsistent contract and file names
## Summary

Inconsistent names between the contract file name and the contract name.

## Vulnerability Details

As per the official Solidity documentation, it is recommended to have the same name for the file and the contract contained in it: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#contract-and-library-names

`Contract and library names should also match their filenames.`

## Impact

The current name structure can be misleading.

## Proof of Concept

Affected contract:
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/interfaces/IAaveLendingPoolV2.sol#L8

## Tools Used

Manual review.

## Recommended Mitigation Steps

Rename the interface or change the file name.

---

# Double import
## Summary

In the `RangeManager.sol` file, `SafeERC20` is imported twice.

## Vulnerability Details

As said in the summary, the `SafeERC20` is imported twice.

## Impact

It is useless and misleading to import the same import in the same contract twice.

## Proof of Concept

- Import 1: https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/RangeManager.sol#L5
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/contracts/RangeManager.sol#L7

## Tools Used

Manual review.

## Recommended Mitigation Steps

Remove one of the duplicated imports:
```diff
import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";
import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";
import "./openzeppelin-solidity/contracts/utils/cryptography/ECDSA.sol";
- import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";
import "./openzeppelin-solidity/contracts/security/ReentrancyGuard.sol";
import "../interfaces/AggregatorV3Interface.sol";
import "../interfaces/ILendingPoolAddressesProvider.sol";
import "../interfaces/IAaveLendingPoolV2.sol";
import "../interfaces/IAaveOracle.sol";
import "../interfaces/IUniswapV2Pair.sol";
import "../interfaces/IUniswapV2Factory.sol";
import "../interfaces/IUniswapV2Router01.sol";
import "../interfaces/ISwapRouter.sol";
import "../interfaces/INonfungiblePositionManager.sol";
import "./TokenisableRange.sol";
import "./openzeppelin-solidity/contracts/proxy/beacon/BeaconProxy.sol";
import "./openzeppelin-solidity/contracts/access/Ownable.sol";
import {IPriceOracle} from "../interfaces/IPriceOracle.sol";
```

---

# Interfaces only used in tests should be separated from core interfaces
## Summary

In the project, there are several interfaces only used in the tests. Those interfaces should not be in the same place as the interfaces used in the protocol contracts since they could be misleading.

## Vulnerability Details

In the `contracts/interfaces` folder there are interfaces only used in tests and also interfaces used in the protocol contracts. Having them mixed together can be misleading and confusing so the ideal thing should be to separate them in different folders.

## Impact

This can cause confusion and even use the wrong interfaces in tests or core protocol contracts.

## Proof of Concept

Interfaces only used in tests that should be moved to another folder:
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/interfaces/IAToken.sol#L9
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/interfaces/ICreditDelegationToken.sol#L4
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/interfaces/IERC20.sol#L9
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/interfaces/IInitializableAToken.sol#L12
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/interfaces/ILendingPoolConfigurator.sol#L5
- https://github.com/code-423n4/2023-08-goodentry/blob/4b785d455fff04629d8675f21ef1d1632749b252/interfaces/IScaledBalanceToken.sol#L4C11-L4C30

## Tools Used

Manual review.

## Recommended Mitigation Steps

Separate the test interfaces and the protocol interfaces in different folders.