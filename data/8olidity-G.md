
## The same calculation can be defined as a variable

`fee0 * lp / totalSupply()` and `fee1 * lp / totalSupply()` are the same calculation, and the calculation result can be represented by a variable
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L316-L323
```
    if (fee0 > 0) {
      TOKEN0.token.safeTransfer( msg.sender, fee0 * lp / totalSupply());//@audit 
      removed0 += fee0 * lp / totalSupply();
      fee0 -= fee0 * lp / totalSupply();
    } 
    if (fee1 > 0) {
      TOKEN1.token.safeTransfer(  msg.sender, fee1 * lp / totalSupply());
      removed1 += fee1 * lp / totalSupply();
      fee1 -= fee1 * lp / totalSupply();
    }
```