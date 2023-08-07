# GAS OPTIMIZATIONS

| Issue Count | Issues | Instances | Gas Saves |
|----------|----------|----------|----------|
| [G-1]  |Using ``calldata`` instead of memory for read-only arguments in external functions saves gas  | 7   | 1974 |
| [G-2]  | State variables can be packed into fewer storage slots  | 2  | 4000 |
| [G-3]  | Structs can be packed into fewer storage slots  | 2  | 4000  |
| [G-4]  | Using bools for storage incurs overhead   | 3   | 60000   |
| [G-5]  | Functions guaranteed to revert when called by normal users can be marked payable   | 13  | 273  |
| [G-6]  | Multiple accesses of a mapping/array should use a local variable cache  | 10  | 1000  |
| [G-7]  | State variable should be cached inside the loop| 1   | 100  |
| [G-8]  | IF’s/require() statements that check input arguments should be at the top of the function  | 1  | 400 |



## [G-1] Using ``calldata`` instead of memory for read-only arguments in external functions saves gas

### Saves ``1974 GAS``

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

### ``options, amounts, sourceSwap`` can be changed to calldata instead of memory. Because read-only arguments : Saves ``846 GAS``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L159-L165

```diff
FILE: 2023-08-goodentry/contracts/PositionManager/OptionsPositionManager.sol

159: function buyOptions(
160:    uint poolId, 
+ 161:    address[] calldata options, 
- 161:    address[] memory options, 
+ 162:    uint[] calldata amounts, 
- 162:    uint[] memory amounts, 
+ 163:    address[] memory sourceSwap
- 163:    address[] memory sourceSwap
164:  )
165:    external
166:  {

```

### ``options, amounts`` can be changed to ``calldata`` instead of ``memory``. Because read-only arguments: Saves ``564 GAS``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L189-L196

```diff
FILE: 2023-08-goodentry/contracts/PositionManager/OptionsPositionManager.sol

189: function liquidate (
190:    uint poolId, 
191:    address user,
+ 192:    address[] memory options, 
- 192:    address[] memory options, 
+ 193:    uint[] memory amounts,
- 193:    uint[] memory amounts,
194:    address collateralAsset
195:  )
196:    external
197:  {

```

### ``startName, endName`` can be changed to ``calldata`` instead of ``memory``. Because read-only arguments : Saves ``564 GAS``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L75


```diff
FILE: 2023-08-goodentry/contracts/RangeManager.sol

- 75: function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

+ 75: function generateRange(uint128 startX10, uint128 endX10, string calldata startName, string calldata endName, address beacon) external onlyOwner {

```
##

## [G-2] State variables can be packed into fewer storage slots

### Saves ``4000 GAS, 2 SLOT``

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

### ``baseFeeX4`` can be declared uint96 instead of ``uint`` : Saves ``2000 GAS``, ``1 SLOT``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L48-L54

``baseFeeX4`` value is not exceed ``1e4`` because ``require(newBaseFeeX4 < 1e4, "GEV: Invalid Base Fee");``  this check

```diff
FILE: Breadcrumbs2023-08-goodentry/contracts/GeVault.sol

48:  ERC20 public token0;
+ 51:  /// @notice Pool base fee 
+ 52:  uint96 public baseFeeX4 = 20;
49:  ERC20 public token1;
50:  bool public isEnabled = true;
- 51:  /// @notice Pool base fee 
- 52:  uint public baseFeeX4 = 20;
53:  /// @notice Max vault TVL with 8 decimals
54:  uint public tvlCap = 1e12;

```

### ``lowerTick,upperTick,feeTier,liquidity`` can be packed together : Saves ``2000 GAS, 1 SLOT``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L52

```diff
FILE: Breadcrumbs2023-08-goodentry/contracts/TokenisableRange.sol

  int24 public lowerTick;
  int24 public upperTick;
  uint24 public feeTier;
+  uint128 public liquidity;
  
  uint256 public tokenId;
  uint256 public fee0;
  uint256 public fee1;
  
  struct ASSET {
    ERC20 token;
    uint8 decimals;
  }
  
  ASSET public TOKEN0;
  ASSET public TOKEN1;
  IAaveOracle public ORACLE;
  
  string _name;
  string _symbol;
  
  enum ProxyState { INIT_PROXY, INIT_LP, READY }
  ProxyState public status;
  address private creator;
  
-  uint128 public liquidity;
  // @notice deprecated, keep to avoid beacon storage slot overwriting errors
  address public TREASURY_DEPRECATED = 0x22Cc3f665ba4C898226353B672c5123c58751692;
  uint public treasuryFee_deprecated = 20;

```

##

## [G-3] Structs can be packed into fewer storage slots

### Saves ``4000 GAS``, ``2 SLOTs``

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

### ``deadline`` can be downcasted to uint96 instead of uint256 : saves ``4000 GAS, 2 SLOTs``. 

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L22-L31

The current timestamp is 2023-08-07 04:44:39 PST. The maximum timestamp that can be stored in a uint96 is 2**96 - 1, which is about 1.8446744073709552e+18 seconds. This is more than 276 years, so it will take at least 276 years for the block timestamp to overflow when stored in a uint96

```diff
FILE: 2023-08-goodentry/contracts/helper/V3Proxy.sol

10: struct ExactInputSingleParams {
11:        address tokenIn;
12:        address tokenOut;
13:        uint24 fee;
14:        address recipient;
+ 15:        uint96 deadline;
- 15:        uint256 deadline;
16:        uint256 amountIn;
17:        uint256 amountOutMinimum;
18:        uint160 sqrtPriceLimitX96;
19:    }


22: struct ExactOutputSingleParams {
23:        address tokenIn;
24:        address tokenOut;
25:        uint24 fee;
26:        address recipient;
+ 27:        uint96 deadline;
- 27:        uint256 deadline;
28:        uint256 amountOut;
29:        uint256 amountInMaximum;
30:        uint160 sqrtPriceLimitX96;
31:    }

```
##

## [G-4] Using bools for storage incurs overhead

### Saves ``60000 GAS``, ``3 Instances ``

```
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.

```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/GeVault.sol

50: bool public isEnabled = true;

64: bool public baseTokenIsToken0;

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L50

```solidity
FILE: 2023-08-goodentry/contracts/helper/V3Proxy.sol

65: bool acceptPayable;

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L65


##

## [G-5] Functions guaranteed to revert when called by normal users can be marked payable

Saves ``273 GAS``, ``13 Instances ``

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```solidity
101:   function setEnabled(bool _isEnabled) public onlyOwner  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101-L101

```solidity
108:   function setTreasury(address newTreasury) public onlyOwner  
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L108-L108

```solidity
116:   function pushTick(address tr) public onlyOwner  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L116-L116

```solidity
137:   function shiftTick(address tr) public onlyOwner  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L137-L137

```solidity
167:   function modifyTick(address tr, uint index) public onlyOwner  
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L167-L167

```solidity
183:   function setBaseFee(uint newBaseFeeX4) public onlyOwner  
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L183-L183


```solidity
191:   function setTvlCap(uint newTvlCap) public onlyOwner  
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L191-L191

```solidity
94:     function emergencyWithdraw(ERC20 token) onlyOwner external  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L94-L94


```solidity
75:   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner  
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75-L75

```solidity
95:   function initRange(address tr, uint amount0, uint amount1) external onlyOwner 
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L95-L95

```solidity
48:   function deprecatePool(uint poolId) public onlyOwner 
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L48-L48

```solidity

59:   function addPool(
60:     address lendingPoolAddressProvider, 
61:     address token0, 
62:     address token1, 
63:     address ammRouter
64:   ) 
65:     public onlyOwner 
66:     returns (uint poolId)
67:   
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L65-L65

```solidity
24:     function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner 

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24-L24

##

## [G-6] Multiple accesses of a mapping/array should use a local variable cache

### Saves ``1100 GAS``, ``11 SLODs``

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata

### ``ticks[ticks.length-1]``, ``ticks[0]`` should be cached with stack variable : Saves ``200 GAS, 2 SLOD``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L124-L128

```diff
FILE: Breadcrumbs2023-08-goodentry/contracts/GeVault.sol

122: else {
123:      // Check that tick is properly ordered
+   TokenisableRange ticks_ = ticks[ticks.length-1];
124:      if (baseTokenIsToken0) 
- 125:        require( t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap");
+ 125:        require( t.lowerTick() > ticks_.upperTick(), "GEV: Push Tick Overlap");
126:      else 
- 127:        require( t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap");
+ 127:        require( t.upperTick() < ticks_ .lowerTick(), "GEV: Push Tick Overlap");
128:      
129:      ticks.push(TokenisableRange(tr));


145: else {
146:      // Check that tick is properly ordered
+   TokenisableRange ticks_ = ticks[0];
147:      if (!baseTokenIsToken0) 
- 148:        require( t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");
+ 148:        require( t.lowerTick() > ticks_.upperTick(), "GEV: Shift Tick Overlap");
149:      else 
- 150:        require( t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");
+ 150:        require( t.upperTick() < ticks_.lowerTick(), "GEV: Shift Tick Overlap");

```

###  ``tokenisedTicker[step]``,``tokenisedRanges[ tokenisedRanges.length - 1 ]``,``tokenisedTicker[step]`` should be cached  : Saves  ``800 GAS``, ``8 SLOD``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L111-L133


```diff
FILE: Breadcrumbs2023-08-goodentry/contracts/RangeManager.sol

+   TokenisableRange  step_ = tokenisedRanges[step] ; 
- 111: trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress).balanceOf(msg.sender);  
+ 111: trAmt = ERC20(LENDING_POOL.getReserveData(address(step_)).aTokenAddress).balanceOf(msg.sender);  
112:    if (trAmt > 0) {       
113:        LENDING_POOL.PMTransfer(
- 114:          LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress,
+ 114:          LENDING_POOL.getReserveData(address(step_)).aTokenAddress,  
115:          msg.sender, 
116:          trAmt
117:        );
- 118:        trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));
+ 118:        trAmt = LENDING_POOL.withdraw(address(step_), type(uint256).max, address(this));
- 119:        tokenisedRanges[step].withdraw(trAmt, 0, 0);
+ 119:        step_.withdraw(trAmt, 0, 0);
- 120:        emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);
+ 120:        emit Withdraw(msg.sender, address(step_), trAmt);
121: }        
122:
+     TokenisableRange  stepTicker_ = tokenisedTicker[step] ; 
- 123:    trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress).balanceOf(msg.sender);
+ 123:    trAmt = ERC20(LENDING_POOL.getReserveData(address(stepTicker_)).aTokenAddress).balanceOf(msg.sender);
124:    if (trAmt > 0) {    
125:        LENDING_POOL.PMTransfer(
- 126:          LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress, 
+ 126:          LENDING_POOL.getReserveData(address(stepTicker_)).aTokenAddress, 
127:          msg.sender, 
128:          trAmt
129:        );
- 130:        uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));
+ 130:        uint256 ttAmt = LENDING_POOL.withdraw(address(stepTicker_), type(uint256).max, address(this));
- 131:        tokenisedTicker[step].withdraw(ttAmt, 0, 0);
+ 131:        stepTicker_.withdraw(ttAmt, 0, 0);
- 132:        emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);
+ 132:        emit Withdraw(msg.sender, address(stepTicker_), trAmt);
133:    }           

```
##

## [G-7] State variable should be cached inside the loop

### Saves ``100 GAS, 1 SLOD`` for each iterations 

State variable should be cached inside the loop if it is accessed multiple times in the loop. This is because accessing a state variable from memory is a relatively expensive operation, so caching the variable in a local variable will improve performance 

### ``stepList[i]`` should be cached : Saves ``100 GAS, 1 SLOT`` for every iterations 

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L62-L65

```diff
FILE: 2023-08-goodentry/contracts/RangeManager.sol

62: for (uint i = 0; i < len; i++) {
+   Step stepList_ = stepList[i];
- 63:      if (start >= stepList[i].end || end <= stepList[i].start) {
+ 63:      if (start >= stepList_.end || end <= stepList_.start) {
64:        continue;
65:      }


```

##

## [G-8] IF’s/require() statements that check input arguments should be at the top of the function

### Saves ``400 GAS``

FAIL CHEEPLY INSTEAD OF COSTLY 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

### Function parameters should be checked top of the function : Saves ``400 GAS``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L249-L252

```diff
FILE: 2023-08-goodentry/contracts/GeVault.sol

+ 252:    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");
249:    require(isEnabled, "GEV: Pool Disabled");
250:    require(poolMatchesOracle(), "GEV: Oracle Error");
251:    require(token == address(token0) || token == address(token1), "GEV: Invalid Token");
- 252:    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");

```






