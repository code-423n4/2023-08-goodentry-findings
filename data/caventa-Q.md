See TokenisableRange#initProxy

```solidity
1.

_name     = string(abi.encodePacked("Ticker ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));

2.

_symbol    = string(abi.encodePacked("T-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));

3.

_name     = string(abi.encodePacked("Ranger ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));

4.

_symbol   = string(abi.encodePacked("R-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));
```

It is not recommend to encodePacked two strings as string is dynamic type

See the following comment in https://docs.soliditylang.org/en/v0.8.11/abi-spec.html

If you use keccak256(abi.encodePacked(a, b)) and both a and b are dynamic types, it is easy to craft collisions in the hash value by moving parts of a into b and vice-versa. More specifically, abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c"). If you use abi.encodePacked for signatures, authentication or data integrity, make sure to always use the same types and check that at most one of them is dynamic. Unless there is a compelling reason, abi.encode should be preferred.

Change

abi.encodePacked to abi.encode