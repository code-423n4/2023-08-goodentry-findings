
[G-1] sqrt using Babylonian method in TokenisableRange has a much optimized approach

```solidity
  function sqrt(uint x) internal pure returns (uint y) {
      uint z = (x + 1) / 2;
      y = x;
      while (z < y) {
          y = z;
          z = (x / z + z) / 2;
      }
  }
```

## Recommended Mitigation Steps
Consider using the optimized version from Uniswap.
https://github.com/Uniswap/solidity-lib/blob/master/contracts/libraries/Babylonian.sol