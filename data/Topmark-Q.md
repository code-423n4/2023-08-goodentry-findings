1. wrong comment description - from the GeVault.sol contract 
The Comment: [L443](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L443) 
```solidity
443.  /// @dev Simple linear model: from baseFeeX4 / 2 to baseFeeX4 * 2
```
The Application: [L457](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L457) 
```solidity
457. if (adjustedBaseFeeX4 > baseFeeX4 * 3 / 2) adjustedBaseFeeX4 = baseFeeX4 * 3 / 2;
```
Correct comment - 
```solidity
443.  /// @dev Simple linear model: from baseFeeX4 / 2 to baseFeeX4 * 3/2 <--- 3/2 not 2
```

2. Validation Check is Needed for token0 and token1 assignment in the GeVault.sol contract constructor at [L84-L85](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L84-L85) as these two tokens are very sensitive, and values such as name, decimal length, address, not just only against address(0) needs to be validated as many factors and calculations depends solely on the correctness of the properties of these addresses