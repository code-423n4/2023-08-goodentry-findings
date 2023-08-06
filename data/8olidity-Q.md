## Use SafeCast to safely downcast variables

Downcasting int/uints in Solidity can be unsafe due to the potential for data loss and unintended behavior. When downcasting a larger integer type to a smaller one (e.g., uint256 to uint128), the value may exceed the range of the target type, leading to truncation and loss of significant digits. This data loss can result in unexpected state changes, incorrect calculations, or other contract vulnerabilities, ultimately compromising the contracts functionality and reliability. To prevent these risks, developers should carefully consider the range of values their variables may hold and ensure that proper checks are in place to prevent out-of-range values before performing downcasting. Also consider using OZ SafeCast functionality.

```solidity
// https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L304
liquidity = uint128(uint256(liquidity) - removedLiquidity);
```