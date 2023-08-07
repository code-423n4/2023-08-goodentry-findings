Note: Below analysis report is more relevant to code base quality analysis.

### Findings Analysis 1- Conflicted wording in comments referencing old protocol that are mixed with concept introduced in the new protocol reduce code readability

(1) Through out the code base, Roe Lending pool is often referred in code documentations while Roe doesn’t exist in the Good Entry protocol doc.  OptionsPositionManager.sol

(2) UniswapV2Interface is still used in PositionManager.sol, but UniswapV2 is no longer used in the new protocol. 

(3) Vault is named GeVault.sol, but in docs, this Vault is named ezVault.

### Findings Analysis 2- sporadic zero address check.

Some functions do have zero address, while the same variable when modified from a different function doesn’t enforce zero address check.

GeVault.sol-setTreasury(), constructor()

### Findings Analysis 3- TokenisableRange.sol getter functions might return conflicted values could be misleading users

TokenisableRange.sol - getTokenAmountsExcludingFees(), latestAnswer()

getTokenAmounts() is based on uniswap spot price, while latestAnswer() is calculated based on avave oracle price. The token0Amount, and token1Amount calculated in two functions can be very different.

### Time spent:
32 hours