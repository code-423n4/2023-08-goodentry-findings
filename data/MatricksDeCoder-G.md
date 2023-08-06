### GAS-1 Setter functions not checking if new value is different from old value. 

There are opportunities to save gas by ensuring that update functions to state variables only work if the new value being set is different from the old value. This saves the caller some gas of updating a state variable to the same value. 

[GetVault.sol line 101... function setEnabled(bool _isEnabled) ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L101) 
[GetVault.sol line 108... function setTreasury(address newTreasury)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L108) 
[GetVault.sol line 183... function setBaseFee(uint newBaseFeeX4)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L183)
[GetVault.sol line 191...function setTvlCap(uint newTvlCap)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L191)
[RoeRouter.sol line 83 ...function setTreasury(address newTreasury)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RoeRouter.sol#L83)
[FixedOracle.sol line 24 ...function setHardcodedPrice(int256 _hardcodedPrice)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/FixedOracle.sol#L24)


**Impact** -> The above implies that the caller e.g onlyOwner in function setTreasury if they set the new value as the old value they will spend 100 gas according the SSTORE rules, but they have not changed any value. 

**Recommendation** -> It is recommended to first check if the old value differs from new value and only change value if different. See example below by implementing some if checks in these setter functions 
```
 function setTreasury(address newTreasury) public onlyOwner {
    require(newTreasury != address(0x0), "Invalid address");
    if(treasury != newTreasury) {
       treasury = newTreasury;
    }
    emit UpdateTreasury(newTreasury);
  }
```

### GAS-2 Use calldata for arrays instead of memory for external function arguments, parameters

There are many instances where location memory is used when calldata could have been used e. g
address[] memory options vs address[] calldata options 

[OptionsPositionManager.sol line 161... function buyOptions(address[] memory options,)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L161)
[OptionsPositionManager.sol line 162... function buyOptions(uint[] memory amounts,)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L162)
[OptionsPositionManager.sol line 163... function buyOptions(address[] memory sourceSwap,)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L163)
[OptionsPositionManager.sol line 192... function liquidate (address[] memory options, )](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L192)
[OptionsPositionManager.sol line 193... function liquidate (uint[] memory amounts, )](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L193)

**Impact** -> memory arrays when called externally result in abi.decode() using a for-loop to copy each index of the calldata to memoryindex. This is costly as each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly does not need this looping 

**Recommendation** -> It is recommended to change all relevant instances from memory to calldata e.g
```
OptionsPositionManager.sol line 161... function buyOptions(address[] calldata options,)
``` 

### GAS-3 Bools more expensive than uint's for storage variables

There are instances were gas savings can be made by not using bools

[GetVault.sol line 64 ...bool public baseTokenIsToken0;](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L64)
[V3Proxy.sol line 64 ...bool acceptPayable;](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L65)
[RoeRouter.sol line28 ...bool isDeprecated;](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RoeRouter.sol#L28)

 **Impact** -> bools are more expensive than uints because the write operations emits extra SLOAD that firstly reads the slot contents and then goes on to replace the bits the boolean has taken up, it then proceeeds to write this back.

**Recommendation** -> e.g use uint(0) false uint(1) true or uint(1) false uint(2) false instead 

### GAS-4 Access controlled functions guaranteed to revert if not called by those allowed can be marked as payable 

The following are some of the functions that could benefit from marking 'payable' gas savings

[GetVault.sol line 101 ...function setEnabled(bool _isEnabled) public onlyOwner  ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L101)
[GetVault.sol line 108 ...function setTreasury(address newTreasury) public onlyOwner ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L108)
[GetVault.sol line 116 ...function pushTick(address tr) public onlyOwner](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L116)
[GetVault.sol line 137 ...function shiftTick(address tr) public onlyOwner  ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L137)
[GetVault.sol line 167 ...function modifyTick(address tr, uint index) public onlyOwner](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L167)
[GetVault.sol line 183 ...function setBaseFee(uint newBaseFeeX4) public onlyOwner  ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L183)
[GetVault.sol line 191 ...function setTvlCap(uint newTvlCap) public onlyOwner { ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L191)
[V3Proxy.sol line 94 ...function emergencyWithdraw(ERC20 token) onlyOwner external](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L94)
[RangeManager.sol line 95 ...function initRange(address tr, uint amount0, uint amount1) external onlyOwner ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L95)
[RoeRouter.sol line 48 ... function deprecatePool(uint poolId) public onlyOwner](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RoeRouter.sol#L48)
[RoeRouter.sol line 59 ... function addPool(...) public onlyOwner](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RoeRouter.sol#L59)
[RoeRouter.sol line 83 ...function setTreasury(address newTreasury) public onlyOwner ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RoeRouter.sol#L83)
[FixedOracle.sol line 24 ...function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/FixedOracle.sol#L24)

**Impact** -> Making these functions payable will save gas for the allowed caller e.g onlyOwner as compiler will not include checks for whether a payment was provided avoiding the following OPCCODES for sending in value checks --> CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function,

**Recommendation** -> It is recommended to mark all these functions as payable 

### GAS-5 Avoid emitting storage variables 

[GetVault.sol line 362 ...emit Rebalance(tickIndex);](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L362) //instead emit passed in value newTickIndex 
```
tokenisedRanges[ tokenisedRanges.length - 1 ].initProxy(...
tokenisedTicker[ tokenisedTicker.length - 1 ].initProxy(...
emit AddRange(startX10, endX10, tokenisedRanges.length - 1);
```
[RangeManager.sol line 87 ...emit AddRange(startX10, endX10, tokenisedRanges.length - 1);](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L87) // cache array tokenisedRanges length in memory instead and use this value which is being re-read in the function parts example above code and then being reread again in event 

RangeManager.sol from line 107 function removeStep(...)
```
TokenisableRange [] public tokenisedRanges;
trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));
uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step])
emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);
emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);
```
[RangeManager.sol line 107 - 134](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L107) //tokenisedTicker[step] state variables tokenisedTicker[step] and tokenisedRanges[step] not cached as they are reread several times in function and then the events emit them by rereading them directly from storage again 

**Impact** - rereading from storage increases gas costs as SLOAD many multiple times more costly than MLOAD the value that could have been cached from memory or from the value in memory from inputs to functions 
[See Past Code4rena Audit Report "avoid emitting storage values" G-05](https://github.com/ajna-finance/audits/blob/main/code4rena/2023-05-code-arena-report.md)

**Recommendation** - not emit storage variables in events 

### GAS-6 Use do while loops instead of for loops 

It may be ideal to use do while loops as cumulative savings may increase given the number of instances of usages of for loops 
[See Past Code4rena Audit Report "Use do while loops instead of for loops" G-09](https://github.com/ajna-finance/audits/blob/main/code4rena/2023-05-code-arena-report.md)

[PositionManager.sol line 71 ..for ( uint8 k = 0; k<assets.length; k++)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L71)
[PositionManager.sol line 93 ...for ( uint8 k =0; k<assets.length; k++)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L93)
[PositionManager.sol line 172 ... for (uint8 i = 0; i< options.length; )](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L172)
[PositionManager.sol line 203 ...for (uint8 i = 0; i< options.length; )](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L203)
[GeVault.sol line 154 ...for (uint k = 0; k < ticks.length - 2; k++)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L154)
[GeVault.sol line 299 ...for (uint k = 0; k < ticks.length; k++)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L299)
[GeVault.sol line 314 ...for (uint k = 0; k < ticks.length; k++)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L314)
[GeVault.sol line 431 ...for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L431)
[RangeManager.sol line 62 ...for (uint i = 0; i < len; i++) {](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L62)

**Impact** -> do while loop will cost less gas since the condition is not being checked for the first iteration.

**Recommendation** -> change all relevant for loops to do while loops 
```
\\change from 
for (uint i= 0; i<num; i++) { ...doStuff}
\\ change to form
uint i;
do { ...doStuff}
while(i < num)
```

