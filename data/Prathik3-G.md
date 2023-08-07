## Gas Report

### [G-01] - Use revert require statements.
Try using revert and custom errors rather than require statemnets in all the files to save gas.

### [G-02] - Repetition of code.
Try to use the same code wherever possible and do not rewrite it.
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/lib/Sqrt.sol#L8C2-L15C4

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L66C3-L73C4

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/LPOracle.sol#L44C2-L51C4

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

### [G-03] - Explicit assingment of variables.
There are instances of assingning default values to variables.Do not assign default values as it will cost more gas.

1. OptionsPositionManager.sol

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L93
```
    for ( uint8 k =0; k<assets.length; k++){
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L172
```
    for (uint8 i = 0; i< options.length; ){
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L203
```solidity
    for (uint8 i = 0; i< options.length; ){
```

2. GeVault.sol

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L154
```solidity
   for (uint k = 0; k < ticks.length - 2; k++) 
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L299
```solidity
   for (uint k = 0; k < ticks.length; k++){
```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L154
```solidity
   for (uint k = 0; k < ticks.length - 2; k++) 
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L431
```solidity
   for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){
```

