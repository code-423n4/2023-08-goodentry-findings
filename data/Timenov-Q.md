## Summary
I-01 Contract missing `SPDX-License-Identifier`.
I-02 Remove whitespaces for better code readability.
I-03 Same validations, but with different error message.
I-04 Variables not used.

### [I-01] Contract missing `SPDX-License-Identifier`.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol

### [I-02] Remove whitespaces for better code readability.
In a lot of places there are whitespaces, which can be removed to improve the code readability.

For example:
```solidity
if ( amtA < token0Amount ){
```

### [I-03] Same validations, but with different error message.
The validation is the same, but the error message is different. Consider using the same error message.

```solidity
require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");
```

```solidity
require(options.length == amounts.length, "ARRAY_LEN_MISMATCH");
```

### [I-04] Variables not used.

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L343

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L41-L42

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L392