### [QA-001] Reuse once calculated result

`fee0 * lp / totalSupply()` is calculated three times.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L316-L318
```
if (fee0 > 0) {
  TOKEN0.token.safeTransfer( msg.sender, fee0 * lp / totalSupply());
  removed0 += fee0 * lp / totalSupply();
  fee0 -= fee0 * lp / totalSupply();
} 
```

The same occurs for `fee1 > 0`.

### [QA-002] Error in comment explaining math behind `TokenisableRange#deposit`

Error in comment explaining math behind `TokenisableRange#deposit`.


https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L236
```
// DeMorgan: !( (n0 == 0 && fee0 > 0) || (n1 == 0 && fee1 > 0) ) = !(n0 == 0 && fee0 > 0) && !(n0 == 0 && fee1 > 0)
```

Should be:
```
// DeMorgan: !( (n0 == 0 && fee0 > 0) || (n1 == 0 && fee1 > 0) ) = !(n0 == 0 && fee0 > 0) && !(n1 == 0 && fee1 > 0)
```
