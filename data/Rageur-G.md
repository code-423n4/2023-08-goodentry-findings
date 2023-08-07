## GAS-1: Caching global variables is more expensive than using the actual variable

### Description

 It’s cheaper to use global variable as compared to caching.

### Affected file

* FixedOracle.sol (Line: 16, 42, 43)
* GeVault.sol (Line: 259)
* TokenisableRange.sol (Line: 88)

## GAS-2: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* GeVault.sol (Line: 120, 120, 121, 141, 141, 142, 170, 170, 203, 205, 215, 218, 230, 239, 249, 250, 252, 252, 255, 281, 386, 463)
* LPOracle.sol (Line: 29, 29, 63)
* OptionsPositionManager.sol (Line: 69, 91, 167, 198, 235, 261, 275, 283, 310, 393, 420, 442, 444)
* OracleConvert.sol (Line: 23, 23, 44)
* PositionManager.sol (Line: 81, 96, 125)
* RangeManager.sol (Line: 112, 124)
* V3Proxy.sol (Line: 87, 91, 99, 106, 113, 125, 138, 139, 148, 149, 161, 162, 179, 180)

## GAS-3: Functions guaranteed to revert when called by normal users can be marked payable

### Description

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

### Affected file

* FixedOracle.sol (Line: 24)
* GeVault.sol (Line: 101, 108, 116, 137, 167, 183, 191)
* RangeManager.sol (Line: 75, 95)
* RoeRouter.sol (Line: 48, 59, 83)
* V3Proxy.sol (Line: 94)

## GAS-4: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* FixedOracle.sol (Line: 10)

## GAS-5: Require()/Revert() strings longer than 32 bytes cost extra gas

### Description

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas.

### Affected file

* FixedOracle.sol (Line: 11)

## GAS-6: Splitting require() statements that use && saves gas

### Description

Saves 16 gas per instance. If you’re using the Optimizer at 200, instead of using the && operator in a single require statement to check multiple conditions, multiple require statements with 1 condition per require statement should be used to save gas.

### Affected file

* GeVault.sol (Line: 120, 141, 170)
* LPOracle.sol (Line: 29, 101)
* OptionsPositionManager.sol (Line: 167, 344, 495, 523, 536)
* OracleConvert.sol (Line: 23)
* RangeManager.sol (Line: 108)
* RoeRouter.sol (Line: 68)
* TokenisableRange.sol (Line: 209, 271)

## GAS-7: Use ```assembly``` to write address storage values

### Affected file

* FixedOracle.sol (Line: 16, 17, 25)
* GeVault.sol (Line: 84, 85, 87, 88, 89, 90, 91, 92, 102, 109, 185, 192, 361)
* LPOracle.sol (Line: 30, 31, 32, 33, 34)
* OracleConvert.sol (Line: 24, 25)
* PositionManager.sol (Line: 31)
* RangeManager.sol (Line: 50, 51, 52)
* RoeRouter.sol (Line: 35, 85)
* TokenisableRange.sol (Line: 88, 89, 90, 99, 100, 103, 106, 107, 108, 109, 111, 112, 113, 114, 115, 117, 118, 137, 183, 184, 212, 279, 304)
* V3Proxy.sol (Line: 76, 77, 78, 153, 155, 171, 173, 189, 191)

## GAS-8: Use assembly to emit events

### Description

We can use assembly to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs.
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

### Affected file

* FixedOracle.sol (Line: 26)
* GeVault.sol (Line: 103, 110, 131, 160, 172, 186, 193, 240, 283, 362)
* OptionsPositionManager.sol (Line: 106, 148, 240, 323, 401, 455)
* RangeManager.sol (Line: 87, 120, 132, 164)
* RoeRouter.sol (Line: 50, 78, 86)
* TokenisableRange.sol (Line: 119, 162, 214, 284, 326)
* V3Proxy.sol (Line: 121, 134, 143, 157, 175, 193)

## GAS-9: Use calldata instead of memory for function parameters

### Description

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

### Affected file

* GeVault.sol (Line: 67, 67)
* OptionsPositionManager.sol (Line: 61, 159, 159, 159, 189, 189, 470, 491)
* RangeManager.sol (Line: 75, 75)
* TokenisableRange.sol (Line: 85, 85)

## GAS-10: Use constants instead of type(uintx).max

### Description

It uses more gas in the distribution process and also for each transaction than constant usage.

### Affected file

* GeVault.sol (Line: 386)
* OracleConvert.sol (Line: 79)
* PositionManager.sol (Line: 81)
* RangeManager.sol (Line: 118, 130)
* TokenisableRange.sol (Line: 172, 173)

## GAS-11: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* GeVault.sol (Line: 262, 302, 324, 330, 339, 340, 386, 409, 412, 423)
* OptionsPositionManager.sol (Line: 92, 103, 104, 105, 176, 207, 275, 310, 312, 313, 369, 479, 496, 501)
* PositionManager.sol (Line: 81, 90, 115, 127)
* RangeManager.sol (Line: 96, 98, 101, 118, 130, 155, 160, 191, 192)
* TokenisableRange.sol (Line: 138, 139, 153, 159, 160, 171, 228, 229)
* V3Proxy.sol (Line: 95, 115, 127, 132, 164, 167, 182, 186)

## GAS-12: Using immutable on variables that are only set in the constructor and never after (2.1k gas per var)

### Description

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

### Affected file

* FixedOracle.sol (Line: 7)
* GeVault.sol (Line: 48, 49, 59, 60, 61, 63, 64)
* PositionManager.sol (Line: 23)
* RangeManager.sol (Line: 27, 32, 33)

## GAS-13: Using private rather than public for constants saves gas

### Description

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

### Affected file

* GeVault.sol (Line: 62)
* RangeManager.sol (Line: 36)
* TokenisableRange.sol (Line: 58, 59, 60, 61)

## GAS-14: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* V3Proxy.sol (Line: 156, 174, 192)

## GAS-15: abi.encode() is less efficient than abi.encodePacked()

### Description

Use abi.encodePacked() where possible to save gas.

### Affected file

* OptionsPositionManager.sol (Line: 168, 199)