# GOOD ENTRY GAS OPTIMIZATIONS


## [G-01] Use the existing Local variable/global variable when equal to a state variable to avoid reading from state


### Instances
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L93
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L95
```solidity
file: contracts/TokenisableRange.sol

85:   function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {
86:     require(address(_oracle) != address(0x0), "Invalid oracle");
87:     require(status == ProxyState.INIT_PROXY, "!InitProxy");
88:     creator = msg.sender;
89:     status = ProxyState.INIT_LP;
90:     ORACLE = _oracle;
91:    
92:     TOKEN0.token    = asset0;
93:     TOKEN0.decimals = asset0.decimals();
94:     TOKEN1.token     = asset1;
95:     TOKEN1.decimals  = asset1.decimals();
96:    string memory quoteSymbol = asset0.symbol();
97:    string memory baseSymbol  = asset1.symbol();
98:        
99:    int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );
100:    int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );
 .   
 .
 .   
118:   upperTick = _upperTick;
119:    emit InitTR(address(asset0), address(asset1), startX10, endX10);
120:  }
```
In the `initProxy()` function above we could declare new stack variables `token0Decimals` and `token1Decimals` and assign the values of `asset0.decimals()` method and `asset1.decimals()` method respectively to the newly declared stack variables. Then assign the value of `token0Decimals` to `TOKEN0.decimals` and the value of `token1Decimals` to `TOKEN1.decimals`. Then in place of `TOKEN0.decimals` and `TOKEN1.decimals` in the statements `int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) )` and `int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) )` we could use the `token0Decimals` and `token1Decimals` stack variables since they are of same value. Refactoring the code this way would save `SLOAD` `2100` gas units (cold access) for the initial reads of the state variables and a further `SLOAD` `100` gas units (warm access) for subsequent reads. Therefore in all we could save an estimated `4400` gas units by implementing this change. The diff below shows how the code could be refactored:
```diff
diff --git a/contracts/TokenisableRange.sol b/contracts/TokenisableRange.sol     
index 085eb58..2d719a3 100644
--- a/contracts/TokenisableRange.sol
+++ b/contracts/TokenisableRange.sol
@@ -89,15 +89,18 @@ contract TokenisableRange is ERC20("", ""), ReentrancyGuard {
     status = ProxyState.INIT_LP;
     ORACLE = _oracle;

+    uint8 token0Decimals = asset0.decimals();   
+    uint8 token1Decimals = asset1.decimals();   
+
     TOKEN0.token    = asset0;
-    TOKEN0.decimals = asset0.decimals();        
+    TOKEN0.decimals = token0Decimals;
     TOKEN1.token     = asset1;
-    TOKEN1.decimals  = asset1.decimals();       
+    TOKEN1.decimals  = token1Decimals;
     string memory quoteSymbol = asset0.symbol();
     string memory baseSymbol  = asset1.symbol();

-    int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );
-    int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );
+    int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** token1Decimals)) * 1e10 / (uint256(startX10) * 10 ** token0Decimals) ) ) );    
+    int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** token1Decimals)) * 1e10 / (uint256(endX10  ) * 10 ** token0Decimals) ) ) );    

```



## [G-02] Unbounded Gas Consumption Risk Due to External Call Recipients
In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using addr.call{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

### Instances
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156
```solidity
file: contracts/helper/V3Proxy.sol

156:    msg.sender.call{value: msg.value - amounts[0]}("");
```
In the `swapETHForExactTokens()` function a low-level call was made without specifing the amount of gas to be used in the low-level call. This is bad practice as this function could be called by a contract and then the contract's fallback function (if the contract does not specify a receive function) would get executed and could possibly use up all the gas thereby causing the `swapETHForExactTokens()` function to revert and the transaction fails. This scenario could lead to bad consumer experience, you should specify the amount of gas to be used in a low-level call as this would prevent such occurences as only a predefined amount of gas can be used by the calling contract. The code could be refactored to:
```diff
diff --git a/contracts/helper/V3Proxy.sol b/contracts/helper/V3Proxy.sol  
index 45ab57e..2fec342 100644
--- a/contracts/helper/V3Proxy.sol
+++ b/contracts/helper/V3Proxy.sol
@@ -153,7 +153,7 @@ contract V3Proxy is ReentrancyGuard, Ownable {        
         acceptPayable = true;
         ROUTER.refundETH();
         acceptPayable = false;
-        msg.sender.call{value: msg.value - amounts[0]}("");
+        msg.sender.call{value: msg.value - amounts[0], gas: 21000}("");  
         emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L174
```solidity
file: contracts/helper/V3Proxy.sol

174:   payable(msg.sender).call{value: amountOut}("");
```
In the `swapTokensForExactETH()` function a low-level call was made without specifing the amount of gas to be used in the low-level call. This is bad practice as this function could be called by a contract and then the contract's fallback function (if the contract does not specify a receive function) would get executed and could possibly use up all the gas thereby causing the `swapTokensForExactETH()` function to revert and the transaction fails. This scenario could lead to bad consumer experience, you should specify the amount of gas to be used in a low-level call as this would prevent such occurences as only a predefined amount of gas can be used by the calling contract. The code could be refactored to:
```diff
diff --git a/contracts/helper/V3Proxy.sol b/contracts/helper/V3Proxy.sol
index 45ab57e..a3ec5f3 100644
--- a/contracts/helper/V3Proxy.sol
+++ b/contracts/helper/V3Proxy.sol
@@ -171,7 +171,7 @@ contract V3Proxy is ReentrancyGuard, Ownable {
         acceptPayable = true;
         weth.withdraw(amountOut);
         acceptPayable = false;
-        payable(msg.sender).call{value: amountOut}("");
+        payable(msg.sender).call{value: amountOut, gas: 21000}("");
         emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 
     }
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L192
```solidity
192:    payable(msg.sender).call{value: amounts[1]}("");
```
In the `swapExactTokensForETH()` function a low-level call was made without specifing the amount of gas to be used in the low-level call. This is bad practice as this function could be called by a contract and then the contract's fallback function (if the contract does not specify a receive function) would get executed and could possibly use up all the gas thereby causing the `swapExactTokensForETH()` function to revert and the transaction fails. This scenario could lead to bad consumer experience, you should specify the amount of gas to be used in a low-level call as this would prevent such occurences as only a predefined amount of gas can be used by the calling contract. The code could be refactored to:
```diff
diff --git a/contracts/helper/V3Proxy.sol b/contracts/helper/V3Proxy.sol
index 45ab57e..fb37c00 100644
--- a/contracts/helper/V3Proxy.sol
+++ b/contracts/helper/V3Proxy.sol
@@ -189,7 +189,7 @@ contract V3Proxy is ReentrancyGuard, Ownable {      
         acceptPayable = true;
         weth.withdraw(amounts[1]);
         acceptPayable = false;
-        payable(msg.sender).call{value: amounts[1]}("");
+        payable(msg.sender).call{value: amounts[1], gas: 21000}("");
         emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);
     }
```


## [G-03]  Move lesser gas costing require checks to the top
Require() or revert() statements that check input arguments or cost lesser gas should be at the top of the function.
Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.

### Instances
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L249-#L252
```solidity
file: contracts/GeVault.sol

247:  function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity) 
248:  {
249:    require(isEnabled, "GEV: Pool Disabled");
250:    require(poolMatchesOracle(), "GEV: Oracle Error");  @audit this would be refactored to a modifier
251:    require(token == address(token0) || token == address(token1), "GEV: Invalid Token");
252:    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");
.
.
.       
284:  }
```
In the `deposit()` function above the `require(amount > 0 || msg.value > 0, "GEV: Deposit Zero")` statement should be moved to the top of the function since it is the cheapest of the require statement so that in scenarios that the conditions `amount > 0 || msg.value > 0` results to false the function could revert without performing the other more gas consuming operations. The code should be refactored as shown in the diff below:
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..40a86c1 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol
@@ -246,10 +246,11 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {
   /// @param amount Amount of token deposited
   function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity) 
   {
+    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");
     require(isEnabled, "GEV: Pool Disabled");
-    require(poolMatchesOracle(), "GEV: Oracle Error");
+    require(poolMatchesOracle(), "GEV: Oracle Error");  
     require(token == address(token0) || token == address(token1), "GEV: Invalid Token");
-    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");
``` 



## [G-04] Refactor external/internal function to avoid unnecessary SLOAD
The functions below read storage slots that are previously read in the functions that invoke them. We can refactor the external/internal functions to pass cached storage variables as stack variables and avoid the extra storage reads that would otherwise take place in the internal functions.

### Instances
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L107-#L168
```solidity
file: contracts/RangeManager.sol

107:  function removeFromStep(uint256 step) internal {
108:    require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");
109:    uint256 trAmt;
110:    
111:    trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress).balanceOf(msg.sender);   
112:    if (trAmt > 0) {       
113:        LENDING_POOL.PMTransfer(
114:          LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress, 
115:          msg.sender, 
116:          trAmt
117:        );
118:        trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));
119:        tokenisedRanges[step].withdraw(trAmt, 0, 0);
120:        emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);
121:    }        
122:
123:    trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress).balanceOf(msg.sender);
124:    if (trAmt > 0) {    
125:        LENDING_POOL.PMTransfer(
126:          LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress, 
127:          msg.sender, 
128:          trAmt
129:        );
130:        uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));
131:        tokenisedTicker[step].withdraw(ttAmt, 0, 0);
132:        emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);
133:    }           
134:  }
.
.
.
139:  function removeAssetsFromStep(uint256 step) nonReentrant external {
140:    removeFromStep(step);
141:    cleanup();
142:  }
.
.
.
151:  function transferAssetsIntoStep(TokenisableRange tr, uint256 step, uint256 amount0, uint256 amount1) internal {
152:    removeFromStep(step);
153:    if (amount0 > 0) {    
154:      LENDING_POOL.PMTransfer( LENDING_POOL.getReserveData(address(ASSET_0)).aTokenAddress, msg.sender, amount0 );
155:      LENDING_POOL.withdraw( address(ASSET_0), amount0, address(this) );
156:      ASSET_0.safeIncreaseAllowance(address(tr), amount0);
157:    }
158:    if (amount1 > 0) {
159:      LENDING_POOL.PMTransfer( LENDING_POOL.getReserveData(address(ASSET_1)).aTokenAddress, msg.sender, amount1 );
160:      LENDING_POOL.withdraw( address(ASSET_1), amount1, address(this) );
161:      ASSET_1.safeIncreaseAllowance(address(tr), amount1);
162:    }
163:    uint256 lpAmt = tr.deposit(amount0, amount1);
164:    emit Deposit(msg.sender, address(tr), lpAmt);
165:    tr.approve(address(LENDING_POOL), lpAmt);
166:    LENDING_POOL.deposit(address(tr), lpAmt, msg.sender, 0);
167:    cleanup();
168:  }
.
.
.
190:  function cleanup() internal {
191:    uint256 asset0_amt = ASSET_0.balanceOf(address(this));
192:    uint256 asset1_amt = ASSET_1.balanceOf(address(this));
193:    
194:    if (asset0_amt > 0) {
195:      ASSET_0.safeIncreaseAllowance(address(LENDING_POOL), asset0_amt);
196:      LENDING_POOL.deposit(address(ASSET_0), asset0_amt, msg.sender, 0);
197:    }
198:    
199:    if (asset1_amt > 0) {
200:      ASSET_1.safeIncreaseAllowance(address(LENDING_POOL), asset1_amt);
201:      LENDING_POOL.deposit(address(ASSET_1), asset1_amt, msg.sender, 0);
202:    }
203:    
204:    // Check that health factor is not put into liquidation / with buffer
205:    (,,,,,uint256 hf) = LENDING_POOL.getUserAccountData(msg.sender);
206:    require(hf > 1.01e18, "Health factor is too low");
207:  }
```
In the `RangeManager` contract we could refactor the functions `removeFromStep()`, `removeAssetsFromStep()`, `transferAssetsIntoStep()` and `cleanup()` such that we reduce the number of `SLOAD`. The `removeFromStep()` function reads the `LENDING_POOL` state variable but this function was invoked in the `transferAssetsIntoStep()` which also reads the same variable from state. we could refactor the `removeFromStep()` function such that it takes has a `ILendingPool LENDING_POOL_` argument which would be passed to it by any function that invokes it. The diff below shows how the changes could be implemented.
```diff
diff --git a/contracts/RangeManager.sol b/contracts/RangeManager.sol
index 057d17a..0511b81 100644
--- a/contracts/RangeManager.sol
+++ b/contracts/RangeManager.sol

-  function removeFromStep(uint256 step) internal {
+  function removeFromStep(uint256 step, ILendingPool LENDING_POOL_) internal {
     require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");
     uint256 trAmt;
    
-    trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress).balanceOf(msg.sender);
+    trAmt = ERC20(LENDING_POOL_.getReserveData(address(tokenisedRanges[step])).aTokenAddress).balanceOf(msg.sender);   
     if (trAmt > 0) {       
-      LENDING_POOL.PMTransfer(
+      LENDING_POOL_.PMTransfer(
-        LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress,
+        LENDING_POOL_.getReserveData(address(tokenisedRanges[step])).aTokenAddress, 
          msg.sender, 
          trAmt
        );
-      trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));
+      trAmt = LENDING_POOL_.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));
       tokenisedRanges[step].withdraw(trAmt, 0, 0);
       emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);
     }        

-    trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress).balanceOf(msg.sender);
+    trAmt = ERC20(LENDING_POOL_.getReserveData(address(tokenisedTicker[step])).aTokenAddress).balanceOf(msg.sender);
     if (trAmt > 0) {    
-      LENDING_POOL.PMTransfer(
+      LENDING_POOL_.PMTransfer(
-        LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress,
+        LENDING_POOL_.getReserveData(address(tokenisedTicker[step])).aTokenAddress,  
          msg.sender, 
          trAmt
       );
-      uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));
+      uint256 ttAmt = LENDING_POOL_.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));
       tokenisedTicker[step].withdraw(ttAmt, 0, 0);
       emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);
     }           
   }


  function removeAssetsFromStep(uint256 step) nonReentrant external {
-    removeFromStep(step);
-    cleanup();

+    ILendingPool LENDING_POOL_ = LENDING_POOL;
+    removeFromStep(step, LENDING_POOL_);
+    cleanup(LENDING_POOL_);
  }


  function transferAssetsIntoStep(TokenisableRange tr, uint256 step, uint256 amount0, uint256 amount1) internal {
+   ILendingPool  LENDING_POOL_ = LENDING_POOL;

-   removeFromStep(step);
+   removeFromStep(step, LENDING_POOL_);
    if (amount0 > 0) {    
-     LENDING_POOL.PMTransfer( LENDING_POOL.getReserveData(address(ASSET_0)).aTokenAddress, msg.sender, amount0 );
+     LENDING_POOL_.PMTransfer( LENDING_POOL_.getReserveData(address(ASSET_0)).aTokenAddress, msg.sender, amount0 );
-     LENDING_POOL.withdraw( address(ASSET_0), amount0, address(this) );
+     LENDING_POOL_.withdraw( address(ASSET_0), amount0, address(this) );
      ASSET_0.safeIncreaseAllowance(address(tr), amount0);
    }
    if (amount1 > 0) {
-     LENDING_POOL.PMTransfer( LENDING_POOL.getReserveData(address(ASSET_1)).aTokenAddress, msg.sender, amount1 );
+     LENDING_POOL_.PMTransfer( LENDING_POOL_.getReserveData(address(ASSET_1)).aTokenAddress, msg.sender, amount1 );
-     LENDING_POOL.withdraw( address(ASSET_1), amount1, address(this) );
+     LENDING_POOL_.withdraw( address(ASSET_1), amount1, address(this) );
      ASSET_1.safeIncreaseAllowance(address(tr), amount1);
    }
    uint256 lpAmt = tr.deposit(amount0, amount1);
    emit Deposit(msg.sender, address(tr), lpAmt);
-   tr.approve(address(LENDING_POOL), lpAmt);
+   tr.approve(address(LENDING_POOL_), lpAmt);
-   LENDING_POOL.deposit(address(tr), lpAmt, msg.sender, 0);
+   LENDING_POOL_.deposit(address(tr), lpAmt, msg.sender, 0);
-   cleanup();
+   cleanup(LENDING_POOL_);
  }

- function cleanup() internal {
+ function cleanup(ILendingPool LENDING_POOL_) internal {
    uint256 asset0_amt = ASSET_0.balanceOf(address(this));
    uint256 asset1_amt = ASSET_1.balanceOf(address(this));
    
    if (asset0_amt > 0) {
-     ASSET_0.safeIncreaseAllowance(address(LENDING_POOL), asset0_amt);
+     ASSET_0.safeIncreaseAllowance(address(LENDING_POOL_), asset0_amt);
-     LENDING_POOL.deposit(address(ASSET_0), asset0_amt, msg.sender, 0);
+     LENDING_POOL_.deposit(address(ASSET_0), asset0_amt, msg.sender, 0);
    }
    
    if (asset1_amt > 0) {
-     ASSET_1.safeIncreaseAllowance(address(LENDING_POOL), asset1_amt);
+     ASSET_1.safeIncreaseAllowance(address(LENDING_POOL_), asset1_amt);
-     LENDING_POOL.deposit(address(ASSET_1), asset1_amt, msg.sender, 0);
+     LENDING_POOL_.deposit(address(ASSET_1), asset1_amt, msg.sender, 0);
    }
    
    // Check that health factor is not put into liquidation / with buffer
-   (,,,,,uint256 hf) = LENDING_POOL.getUserAccountData(msg.sender);
+   (,,,,,uint256 hf) = LENDING_POOL_.getUserAccountData(msg.sender);
    require(hf > 1.01e18, "Health factor is too low");
  }

```


## [G-05] Avoid using state variable in emit
In scenarios where you have a memory, calldata variable or parameter with the same value as the state variable you should use the memory, calldata variable or parameter in the emit statement rather than state variable.

### Instances
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362
```solidity
file: contracts/GeVault.sol

337:  function deployAssets() internal { 
.
.
.   
361:    if (newTickIndex != tickIndex) tickIndex = newTickIndex;
362:    emit Rebalance(tickIndex);
363:  }
```
We could save 1 `SLOAD` (cold access `2100` gas units) if we emit the stack variable `newTickIndex` in place of the state variable `tickIndex` since the if-statement ` if (newTickIndex != tickIndex) tickIndex = newTickIndex` ensures that always be equal. The diff below shows how the code could be refactored: 
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..6df298a 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol
@@ -357,11 +357,11 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {
       depositAndStash(ticks[tick1Index], 0, availToken1 / 2);
       depositAndStash(ticks[tick1Index+1], 0, availToken1 / 2);
     }
     
     if (newTickIndex != tickIndex) tickIndex = newTickIndex;
-    emit Rebalance(tickIndex);
+    emit Rebalance(newTickIndex);
   }
```



## [G-06] Can transfer 0
In Solidity, performing unnecessary operations can consume more gas than needed, leading to cost inefficiencies. For instance, if a transfer function doesn't have a zero amount check and someone calls it with a zero amount, unnecessary gas will be consumed in executing the function, even though the state of the contract remains the same. By implementing a zero amount check, such unnecessary function calls can be avoided, thereby saving gas and making the contract more efficient.

### Instances
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L139
```solidity
file: contracts/TokenisableRange.sol

134:  function init(uint n0, uint n1) external {
135:    require(status == ProxyState.INIT_LP, "!InitLP");
136:    require(msg.sender == creator, "Unallowed call");
137:    status = ProxyState.READY;
138:    TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);
139:    TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);
.
.
.
163:  }
```
In the `init()` function above we could factor in a check for 0 amount for the arguments `n0` and `n1` before the `TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0)` and `TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1)` statement to eliminate the possibility of performing unnecessary transfer operation that would not change state. The function should be refactored to:
```diff
diff --git a/contracts/TokenisableRange.sol b/contracts/TokenisableRange.sol
index 085eb58..e3b2f88 100644
--- a/contracts/TokenisableRange.sol
+++ b/contracts/TokenisableRange.sol
@@ -132,6 +132,8 @@ contract TokenisableRange is ERC20("", ""), ReentrancyGuard {
   /// @param n1 Amount of base token added
   /// @notice The token amounts must be 95% correct or this will fail the Uniswap slippage check
   function init(uint n0, uint n1) external {
+    require(n0 > 0, "Zero Amount");
+    require(n1 > 0, "Zero Amount");
     require(status == ProxyState.INIT_LP, "!InitLP");
     require(msg.sender == creator, "Unallowed call");
     status = ProxyState.READY;
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L228
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L229
```solidity
file: contracts/TokenisableRange.sol
222:  function deposit(uint256 n0, uint256 n1) external nonReentrant returns (uint256 lpAmt) {
223:    // Once all assets were withdrawn after initialisation, this is considered closed
234:    // Prevents TR oracle values from being too manipulatable by emptying the range and redepositing 
235:    require(totalSupply() > 0, "TR Closed"); 
236:    
237:    claimFee();
238:    TOKEN0.token.transferFrom(msg.sender, address(this), n0);
239:    TOKEN1.token.transferFrom(msg.sender, address(this), n1);
240:    
241:    uint newFee0; 
242:    uint newFee1;
```
In the `deposit()` function above we could factor in a check for 0 amount for the arguments `n0` and `n1` before the `TOKEN0.token.transferFrom(msg.sender, address(this), n0)` and `TOKEN1.token.transferFrom(msg.sender, address(this), n1)` statement to eliminate the possibility of performing unnecessary transfer operation that would not change state. The function should be refactored to:
```diff
diff --git a/contracts/TokenisableRange.sol b/contracts/TokenisableRange.sol     
index 085eb58..e3b2f88 100644
--- a/contracts/TokenisableRange.sol
+++ b/contracts/TokenisableRange.sol
@@ -132,6 +132,8 @@ contract TokenisableRange is ERC20("", ""), ReentrancyGuard {
   /// @param n1 Amount of base token added
   /// @notice The token amounts must be 95% correct or this will fail the Uniswap slippage check
   function init(uint n0, uint n1) external {
+    require(n0 > 0, "Zero Amount");
+    require(n1 > 0, "Zero Amount");
     require(status == ProxyState.INIT_LP, "!InitLP");
     require(msg.sender == creator, "Unallowed call");
     status = ProxyState.READY;
```


## [G-07] Use v4.9.0 and above OpenZeppelin contracts
The v4.9.0 version of OpenZeppelin provides many small gas optimizations which could reduces the gas usage of your contracts. https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v4.9.0


## [G-08] Using bools for storage incurs overhead

### 1 Instance
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L50
```solidity
file: contracts/GeVault.sol

50:   bool public isEnabled = true;
```
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.

```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..94c3558 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol

@@ -45,11 +45,11 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {
   
-  bool public isEnabled = true;
+  uint256 public isEnabled = 2;
   
   
@@ -96,12 +96,16 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {
   
-  function setEnabled(bool _isEnabled) public onlyOwner { 
-    isEnabled = _isEnabled; 
+  function setEnabled(bool _isEnabled) public onlyOwner {
+    if(_isEnabled) 
+      isEnabled = 1;
+    else {
+      isEnabled = 2;
+    } 
     emit SetEnabled(_isEnabled);
   }
   
@@ -200,11 +204,11 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {
   
-    if (isEnabled) deployAssets();
+    if (isEnabled == 1) deployAssets();
   }
   
 
@@ -234,21 +238,21 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {
     
-    if (isEnabled) deployAssets();
+    if (isEnabled == 1) deployAssets();
     emit Withdraw(msg.sender, token, amount, liquidity);
   }
 
 
   /// @notice deposit tokens in the pool, convert to WETH if necessary
   /// @param token Token address
   /// @param amount Amount of token deposited
   function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity) 
   {
-    require(isEnabled, "GEV: Pool Disabled");
+    require(isEnabled == 1, "GEV: Pool Disabled");
```

## [G-09] Duplicated require/if checks should be refactored to a modifier or function
This saves deployment gas cost and also enhances code redability.

### Instances
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L203  
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L215 
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L250
```solidity
file: contracts/GeVault.sol

202:  function rebalance() public {
203:    require(poolMatchesOracle(), "GEV: Oracle Error");  @audit refactor to modifier
204:    removeFromAllTicks();
205:    if (isEnabled) deployAssets();
206:  }

214:  function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {
215:    require(poolMatchesOracle(), "GEV: Oracle Error");  @audit refactor to modifier
216:    if (liquidity == 0) liquidity = balanceOf(msg.sender);
217:    require(liquidity <= balanceOf(msg.sender), "GEV: Insufficient Balance");
218:    require(liquidity > 0, "GEV: Withdraw Zero");
.
.
.    
240:    emit Withdraw(msg.sender, token, amount, liquidity);
241:  }

247:  function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity) 
248:  {
249:    require(isEnabled, "GEV: Pool Disabled");
250:    require(poolMatchesOracle(), "GEV: Oracle Error");  @audit refactor to modifier
251:    require(token == address(token0) || token == address(token1), "GEV: Invalid Token");
252:    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");
.
.    
.    
283:    emit Deposit(msg.sender, token, amount, liquidity);
264:  }
```
In the `GeVault.sol` contract the require statement `require(poolMatchesOracle(), "GEV: Oracle Error")` was used in the `rebalance()`, `withdraw()` and `deposit()` functions. We could reduce the contract deployment cost if this require statement is refactored into a modifier. The diff below shows how this could be done.
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..62a1530 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol
@@ -63,6 +63,10 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {
   
+  modifier oracleMatchesPool() {
+    require(poolMatchesOracle(), "GEV: Oracle Error");
+    _;
+  }

@@ -199,8 +203,7 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {
-  function rebalance() public {
-    require(poolMatchesOracle(), "GEV: Oracle Error");
+  function rebalance() public oracleMatchesPool {
     removeFromAllTicks();
     if (isEnabled) deployAssets();
   }

@@ -211,8 +214,7 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {

-  function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {
-    require(poolMatchesOracle(), "GEV: Oracle Error");
+  function withdraw(uint liquidity, address token) public nonReentrant oracleMatchesPool returns (uint amount) {
     if (liquidity == 0) liquidity = balanceOf(msg.sender);
     require(liquidity <= balanceOf(msg.sender), "GEV: Insufficient Balance");
     require(liquidity > 0, "GEV: Withdraw Zero");


@@ -242,16 +244,16 @@ contract GeVault is ERC20, Ownable, ReentrancyGuard {
 
-  function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity) 
+  function deposit(address token, uint amount) public payable nonReentrant oracleMatchesPool returns (uint liquidity) 
   {
+    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");
     require(isEnabled, "GEV: Pool Disabled");
-    require(poolMatchesOracle(), "GEV: Oracle Error");
     require(token == address(token0) || token == address(token1), "GEV: Invalid Token");
-    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");
+    
     
``` 

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L99
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L106  
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L113  
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L125  
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L138  
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L148  
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L161 
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L179
```solidity
file: contracts/helper/V3Proxy.sol

98:    function getAmountsOut(uint amountIn, address[] calldata path) external returns (uint[] memory amounts) {
99:        require(path.length == 2, "Direct swap only");
100:        amounts = new uint[](2);
101:        amounts[0] = amountIn;
102:        amounts[1] = QUOTER.quoteExactInputSingle(path[0], path[1], feeTier, amountIn, 0);
103:    }


105:    function getAmountsIn(uint amountOut, address[] calldata path) external returns (uint[] memory amounts) {
106:        require(path.length == 2, "Direct swap only");
107:        amounts = new uint[](2);
108:        amounts[0] = QUOTER.quoteExactOutputSingle(path[0], path[1], feeTier, amountOut, 0);
109:        amounts[1] = amountOut;
110:    }


112:    function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {
113:        require(path.length == 2, "Direct swap only");
114:        ERC20 ogInAsset = ERC20(path[0]);
.
.
.
122:    }


124:    function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {
125:        require(path.length == 2, "Direct swap only");
126:        ERC20 ogInAsset = ERC20(path[0]);
.
.
.
134:    }


137:    function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
138:        require(path.length == 2, "Direct swap only");
139:        require(path[0] == ROUTER.WETH9(), "Invalid path");
.
.
.  
144:    }


147:    function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
148:        require(path.length == 2, "Direct swap only");
149:        require(path[0] == ROUTER.WETH9(), "Invalid path");
150:        amounts = new uint[](2);
.
.
. 
158    }


160:    function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
161:        require(path.length == 2, "Direct swap only");
162:        require(path[1] == ROUTER.WETH9(), "Invalid path");
.
.
.         
176:    }


178:    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
179:        require(path.length == 2, "Direct swap only");
180        require(path[1] == ROUTER.WETH9(), "Invalid path");
.
.
.
194:    }
```
In the `V3Proxy.sol` contract the require statement `require(path.length == 2, "Direct swap only")` was used in the `getAmountsOut()`, `getAmountsIn()`, `swapExactTokensForTokens()`, `swapTokensForExactTokens()`, `swapExactETHForTokens()`, `swapETHForExactTokens()`, `swapTokensForExactETH()` and `swapExactTokensForETH()` functions. We could reduce the contract deployment cost if this require statement is refactored into a modifier. The diff below shows how this could be implemented.

```diff
diff --git a/contracts/helper/V3Proxy.sol b/contracts/helper/V3Proxy.sol
index 45ab57e..b0ed557 100644
--- a/contracts/helper/V3Proxy.sol
+++ b/contracts/helper/V3Proxy.sol
@@ -79,6 +79,11 @@ contract V3Proxy is ReentrancyGuard, Ownable {

+    modifier checkPathLength(address[] calldata path) {
+        require(path.length == 2, "Direct swap only");
+        _;
+    }
+
     
@@ -93,38 +98,51 @@ contract V3Proxy is ReentrancyGuard, Ownable {
 
-    function getAmountsOut(uint amountIn, address[] calldata path) external returns (uint[] memory amounts) {
-        require(path.length == 2, "Direct swap only");
+    function getAmountsOut(
+        uint amountIn, 
+        address[] calldata path
+    ) external checkPathLength(path) returns (uint[] memory amounts) {
         amounts = new uint[](2);
         amounts[0] = amountIn;
         amounts[1] = QUOTER.quoteExactInputSingle(path[0], path[1], feeTier, amountIn, 0);
     }
 
-    function getAmountsIn(uint amountOut, address[] calldata path) external returns (uint[] memory amounts) {
-        require(path.length == 2, "Direct swap only");
+    function getAmountsIn(
+        uint amountOut, 
+        address[] calldata path
+    ) external checkPathLength(path) returns (uint[] memory amounts) {
         amounts = new uint[](2);
         amounts[0] = QUOTER.quoteExactOutputSingle(path[0], path[1], feeTier, amountOut, 0);
         amounts[1] = amountOut;
     }
 
-    function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {
-        require(path.length == 2, "Direct swap only");
+    function swapExactTokensForTokens(
+        uint amountIn, 
+        uint amountOutMin, 
+        address[] calldata path, 
+        address to, 
+        uint deadline
+    ) external checkPathLength(path) returns (uint[] memory amounts) {
         ERC20 ogInAsset = ERC20(path[0]);
         ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
         ogInAsset.safeApprove(address(ROUTER), amountIn);
         amounts = new uint[](2);
         amounts[0] = amountIn;         
         amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountIn, amountOutMin, 0));
         ogInAsset.safeApprove(address(ROUTER), 0);
         emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 
     }
 
-    function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {
-        require(path.length == 2, "Direct swap only");
+    function swapTokensForExactTokens(
+        uint amountOut, 
+        uint amountInMax, 
+        address[] calldata path, 
+        address to, 
+        uint deadline) external checkPathLength(path) returns (uint[] memory amounts) {
         ERC20 ogInAsset = ERC20(path[0]);
         ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);
         ogInAsset.safeApprove(address(ROUTER), amountInMax);
         amounts = new uint[](2);
         amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountOut, amountInMax, 0));         
     }
 
-    function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
-        require(path.length == 2, "Direct swap only");
+    function swapExactETHForTokens(
+        uint amountOutMin, 
+        address[] calldata path, 
+        address to, 
+        uint deadline
+    ) payable external checkPathLength(path) returns (uint[] memory amounts) {
         require(path[0] == ROUTER.WETH9(), "Invalid path");
         amounts = new uint[](2);
         amounts[0] = msg.value;         
         amounts[1] = ROUTER.exactInputSingle{value: msg.value}(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, msg.value, amountOutMin, 0));
         emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);  
     }
 
     
-    function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
-        require(path.length == 2, "Direct swap only");
+    function swapETHForExactTokens(
+        uint amountOut, 
+        address[] calldata path, 
+        address to, 
+        uint deadline
+    ) payable external checkPathLength(path) returns (uint[] memory amounts) {
         require(path[0] == ROUTER.WETH9(), "Invalid path");
         amounts = new uint[](2);
         amounts[0] = ROUTER.exactOutputSingle{value: msg.value}(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountOut, msg.value, 0));         
         amounts[1] = amountOut;
         acceptPayable = true;
@@ -155,12 +181,17 @@ contract V3Proxy is ReentrancyGuard, Ownable {
         acceptPayable = false;
         msg.sender.call{value: msg.value - amounts[0]}("");
         emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 
     }
     
-    function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
-        require(path.length == 2, "Direct swap only");
+    function swapTokensForExactETH(
+        uint amountOut, 
+        uint amountInMax, 
+        address[] calldata path, 
+        address to, 
+        uint deadline
+    ) payable external checkPathLength(path) returns (uint[] memory amounts) {
         require(path[1] == ROUTER.WETH9(), "Invalid path");
         ERC20 ogInAsset = ERC20(path[0]);
         ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);
         ogInAsset.safeApprove(address(ROUTER), amountInMax);
         amounts = new uint[](2);
@@ -173,12 +204,17 @@ contract V3Proxy is ReentrancyGuard, Ownable {
         acceptPayable = false;
         payable(msg.sender).call{value: amountOut}("");
         emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 
     }
        
-    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
-        require(path.length == 2, "Direct swap only");
+    function swapExactTokensForETH(
+        uint amountIn, 
+        uint amountOutMin, 
+        address[] calldata path, 
+        address to, 
+        uint deadline
+    ) payable external checkPathLength(path) returns (uint[] memory amounts) {
         require(path[1] == ROUTER.WETH9(), "Invalid path");
         ERC20 ogInAsset = ERC20(path[0]);
         ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
         ogInAsset.safeApprove(address(ROUTER), amountIn);
         amounts = new uint[](2);

```


## [G-10] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.
the recommended Mitigation Steps is that Functions guaranteed to revert when called by normal users can be marked payable.

### 10 Instances
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101
```solidity
file: contracts/GeVault.sol

101:  function setEnabled(bool _isEnabled) public onlyOwner { 
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `setEnabled()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..a05a7fe 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol

-  function setEnabled(bool _isEnabled) public onlyOwner { 
+  function setEnabled(bool _isEnabled) public payable onlyOwner {  
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L108
```solidity
file: contracts/GeVault.sol

108:  function setTreasury(address newTreasury) public onlyOwner { 
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `setTreasury()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..a05a7fe 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol

-    function setTreasury(address newTreasury) public onlyOwner { 
+    function setTreasury(address newTreasury) public onlyOwner { 
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L116
```solidity
file: contracts/GeVault.sol

116:  function pushTick(address tr) public onlyOwner {
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `pushTick()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..a05a7fe 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol

-    function pushTick(address tr) public onlyOwner {
+    function pushTick(address tr) public payable onlyOwner {
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L137
```solidity
file: contracts/GeVault.sol

137:  function shiftTick(address tr) public onlyOwner {
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `shiftTick()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..a05a7fe 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol

-   function shiftTick(address tr) public onlyOwner {
+   function shiftTick(address tr) public payable onlyOwner {
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L167
```solidity
file: contracts/GeVault.sol

167:  function modifyTick(address tr, uint index) public onlyOwner {
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `modifyTick()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..a05a7fe 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol

-   function modifyTick(address tr, uint index) public onlyOwner {
+   function modifyTick(address tr, uint index) public payable onlyOwner {
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L183
```solidity
file: contracts/GeVault.sol

183:  function setBaseFee(uint newBaseFeeX4) public onlyOwner {
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `setBaseFee()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..a05a7fe 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol

-   function setBaseFee(uint newBaseFeeX4) public onlyOwner {
+   function setBaseFee(uint newBaseFeeX4) public payable onlyOwner {
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L191
```solidity
file: contracts/GeVault.sol

191:  function setTvlCap(uint newTvlCap) public onlyOwner {
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `setTvlCap()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/GeVault.sol b/contracts/GeVault.sol
index 4c9dae3..a05a7fe 100644
--- a/contracts/GeVault.sol
+++ b/contracts/GeVault.sol

-    function setTvlCap(uint newTvlCap) public onlyOwner {
+    function setTvlCap(uint newTvlCap) public payable onlyOwner {
```


- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75
```solidity
file: contracts/RangeManager.sol

75:  function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `generateRange()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/RangeManager.sol b/contracts/RangeManager.sol
index 057d17a..613eaa2 100644
--- a/contracts/RangeManager.sol
+++ b/contracts/RangeManager.sol

-  function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {
+  function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external payable onlyOwner {
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L95
```solidity
file: contracts/RangeManager.sol

95:  function initRange(address tr, uint amount0, uint amount1) external onlyOwner {
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `initRange()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/RangeManager.sol b/contracts/RangeManager.sol
index 057d17a..613eaa2 100644
--- a/contracts/RangeManager.sol
+++ b/contracts/RangeManager.sol

-  function initRange(address tr, uint amount0, uint amount1) external onlyOwner {
+  function initRange(address tr, uint amount0, uint amount1) external payable onlyOwner {
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24
```solidity
file: contracts/helper/FixedOracle

24:    function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {
```
We could avoid these opcodes: CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) ( these opcodes cost an average of about `21` gas per call to the function) if we add the `payable` keyword to `setHardcodedPrice()` function header. The  function header could be refactored as shown in the diff below:
```diff
diff --git a/contracts/helper/FixedOracle.sol b/contracts/helper/FixedOracle.sol
index 3916768..171e982 100644
--- a/contracts/helper/FixedOracle.sol
+++ b/contracts/helper/FixedOracle.sol

-    function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {
+    function setHardcodedPrice(int256 _hardcodedPrice) external payable onlyOwner {
```
```
Estimated gas saved: 10 * 21 = 210 gas units
```


## [G-11] Splitting require() statements that use && saves gas - (saves 8 gas per &&) 
Instead of using the && operator in a single require statement to check multiple conditions,using multiple require statements with 1 condition per require statement will save 8 GAS per &&.
The gas difference would only be realized if the revert condition is realized(met).

### 11 Instances
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L167
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L495
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L523
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L536
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L141
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L170
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L209
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L271
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L108
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L101
- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L23

```
Estimated gas saved: 8 * 11 = 88
```
