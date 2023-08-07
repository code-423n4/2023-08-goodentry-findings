## 1. `TokenisableRange.getTokenAmounts(amount)` reverts when totalSupply() is 0

### Codes
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L377-L381

### Recommended Mitigation Steps
```diff
function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){
    + if (totalSupply() == 0) return (0, 0);
    (token0Amount, token1Amount) = getTokenAmountsExcludingFees(amount);
    token0Amount += fee0 * amount / totalSupply();
    token1Amount += fee1 * amount / totalSupply();
}
```