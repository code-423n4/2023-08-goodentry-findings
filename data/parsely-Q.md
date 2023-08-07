# [NC-01] V3Proxy.sol imports and inherits ReentrancyGuard but it it is not used.
## Context
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L60
```
contract V3Proxy is ReentrancyGuard, Ownable {
```
## Description
The V3Proxy contract imports and inherits ReentrancyGuard however it is never used and if the  `is ReentrancyGuard` is removed it still compiles without any issues.

## Resolution
It can be considered to make use of the ReentrancyGuard functionality or remove the inheritance and import.

# [NC-02] Unused local variable : `amount0ft`.
## Context
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L343
```
(uint amount0ft, uint amount1ft) = ticks[newTickIndex].getTokenAmountsExcludingFees(1e18);
```
## Description
The local variable `amount0ft` is not used.

## Resolution
Consider changing the line to:
```
(, uint amount1ft) = ticks[newTickIndex].getTokenAmountsExcludingFees(1e18);
```