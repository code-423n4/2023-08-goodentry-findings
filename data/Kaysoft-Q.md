## [LC-1] Avoid variable shadowing
`GeVault.sol#constructor.name` and `GeVault.sol#constructor.symbol` shadows `ERC20.sol#name` and `ERC20.sol#symbol`.

File: https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L72-L73

#### Recommended mitigation steps
Rename the `name` and `symbol` parameter of the GeVault contstructor

## [NC-1] Add `view` keyword to the `sanityCheckUnderlying` and `addDust` functions of OptionsPositionsManager.sol.

File: https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L533-L551

## [NC-2] Avoid hardcoding addresses 
Avoid hardcoding addresses since the project will be deployed to multiple chains. 
The following are not part of the bot report.
File: 
- https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L54
- https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L60

