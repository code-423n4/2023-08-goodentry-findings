# 1) Add ownership/authorization checks to functions `claimFee` and `withdraw`.
Functions `claimFee` and `withdraw` can be called and manipulated by unauthorized users.
It is also not save because `claimFee` goes into `withdraw` function and affects its execution.

Instance:
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L167
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L292

Recommended mitigation steps:
To reinforce security, consider adding user authorization check or `onlyOwner` modifier for the functions `claimFee` and `withdraw`.