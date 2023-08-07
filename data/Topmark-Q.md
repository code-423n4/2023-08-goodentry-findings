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