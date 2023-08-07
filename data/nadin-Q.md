# QA Report
## Summary
### Total  07 instances
- [01] Faulty function documentation in `INonfungiblePositionManager.sol`
- [02] Should add check and revert if return `zero` in `TokenisableRange#getTokenAmounts()`
- [03] The `sqrt()` function is implemented in `LPOracle.sol` and `TokenisableRange.sol` and `Sqrt.sol` 
- [04] Should implementing the latest openzeppelin to avoid unexpected bugs
- [05] Use SMTChecker
- [06] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
- [07] Include the project name and development team information in the contract to increase the popularity and trust of users in the project

## [01] Faulty function documentation in `INonfungiblePositionManager.sol`
### Description
-The function `decreaseLiquidity()` in the `INonfungiblePositionManager` has a faulty `@param` field.
- The documentation states: [here](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/interfaces/INonfungiblePositionManager.sol#L149)
```
File: INonfungiblePositionManager.sol
149:    /// amount The amount by which liquidity will be decreased,
```
- However, the field in the `DecreaseLiquidityParams` struct is called `liquidity` not `amount` : [here](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/interfaces/INonfungiblePositionManager.sol#L141)
```
File: INonfungiblePositionManager.sol
141:        uint128 liquidity;
```
### Recommendation
Change `amount` to `liquidity` in the param
```
File: INonfungiblePositionManager.sol
--- 149:    /// amount The amount by which liquidity will be decreased,
+++ 149:    /// liquidity The amount by which liquidity will be decreased,
```


## [02] Should add check and revert if return `zero` in `TokenisableRange#getTokenAmounts()`
### Description
- Take a look at [TokenisableRange#getTokenAmounts](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L377-L381)
```
377:  function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){
378:    (token0Amount, token1Amount) = getTokenAmountsExcludingFees(amount);
379:    token0Amount += fee0 * amount / totalSupply(); // @audit can be return 0
380:    token1Amount += fee1 * amount / totalSupply(); // @audit can be return 0
381:  }
```
1. First lack of check `totalSupply()  == 0` 
2. In `L380` and `L381` can be returned to `0` if `totalSupply()` is greater than the `numerator`.
### Recommendation:
1. Add check `if ( totalSupply() == 0 ) return 0;`
2. Should add check and revert if return `zero` in `L380` and `Line381`.

## [03] The `sqrt()` function is implemented in `LPOracle.sol` and `TokenisableRange.sol` and `Sqrt.sol` 
### Description
1. [LPOracle.sol#sqrt()](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/LPOracle.sol#L44-L51)
```
44:  function sqrt(uint x) internal pure returns (uint y) {
45:    uint z = (x + 1) / 2;
46:    y = x;
47:    while (z < y) {
48:      y = z;
49:      z = (x / z + z) / 2;
50:    }
51:  }
```
2. [TokenisableRange.sol#sqrt()](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L66-L73)
```
66:  function sqrt(uint x) internal pure returns (uint y) {
67:      uint z = (x + 1) / 2;
68:      y = x;
69:      while (z < y) {
70:          y = z;
71:          z = (x / z + z) / 2;
72:      }
73:  }
```
3. [library Sqrt](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/lib/Sqrt.sol)
```
8:  function sqrt(uint x) internal pure returns (uint y) {
9:      uint z = (x + 1) / 2;
10:      y = x;
11:      while (z < y) {
12:          y = z;
13:          z = (x / z + z) / 2;
14:      }
```
Implementing a function in all three contracts is wasteful
### Recommendation
Should import `library Sqrt` into `LPOracle.so`l and `TokenisableRange.sol` so that `sqrt()` function can be used.

## [04] Should implementing the latest openzeppelin to avoid unexpected bugs
### Description
See docs here: [@openzeppelin/contracts vulnerabilities](https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts)
`goodentry` is using old openzeppelin version `4.4.1` : [see here](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/openzeppelin-solidity/contracts/package.json#L4)
```
2:  "name": "@openzeppelin/contracts",
3:  "description": "Secure Smart Contract library for Solidity",
4:  "version": "4.4.1",
```
### Recommendation
Update and use the latest openzeppelin `v4.9.3`: [see here](https://github.com/OpenZeppelin/openzeppelin-contracts/releases)

## [05] Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## [06] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONSTANTS
### Description:
There are many addresses and constants used in the system. It is recommended to put the most used ones in one file (for example constants.sol, use inheritance to access these values).

This will help with readability and easier maintenance for future changes. This also helps with any issues, as some of these hard-coded values are admin addresses.
#### constants.sol
Use and import this file in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

## [07] Include the project name and development team information in the contract to increase the popularity and trust of users in the project
### Recommendation: Use form like FraxFinance project
https://github.com/FraxFinance/frxETH-public/blob/7f7731dbc93154131aba6e741b6116da05b25662/src/sfrxETH.sol#L4-L24