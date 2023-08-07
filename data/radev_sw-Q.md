In the [GetVault#deposit()](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L247-L284) function on line 269 exist this check:

`require(tvlCap > valueX8 + getTVL(), "GEV: Max Cap Reached");`

However the require validation should be:
`require(tvlCap >= valueX8 + getTVL(), "GEV: Max Cap Reached");`