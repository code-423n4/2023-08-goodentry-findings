# Gas Optimization
### Instance found 
46

### Finding
**To save gas usage and limit, use custom error instead of require. you can use custom errors to reduce both deployment and runtime gas costs. In addition, they are very convenient as you can easily pass dynamic information to them. **

### Resolution
The line of code *require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");* in the *OptionsPositionManager.sol line 69 and 91* could be written as **if(address(lendingPool) != msg.sender) revert CallUnallowed();** where *error callUnallowed();* has to be stated in the code.

### Other Findings where the custom error might be necessary to save gas

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol]
* Lines: 137, 167, 198, 261, 393, 420, 441, 536*

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol]
* Line: 30

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol]
* Lines: 87, 91, 99, 106, 113, 125, 138, 139, 148, 149, 161, 162, 179, 180*

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol]
* Lines: 79-81, 120, 141, 170, 203, 215, 249-251, 256*

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol]
* Lines: 49 and 76*

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol]
* Lines: 34, 68, and 84*

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol]
* Lines: 86, 87, 136 and 136*

