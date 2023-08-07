# gas 

# summary 
|      | issue | instance  |
|------|-------|-----------|
|[G-01]| Use calldata instead of memory for function arguments that do not get mutated|2|
|[G-02]|Avoid contract existence checks by using low level calls|11|
|[G-03]|Use do while loops instead of for loops|2|
|[G-04]|Use assembly to perform efficient back-to-back calls|16|
|[G-05]|Use assembly to emit an event|16|
|[G-06]|Avoid emitting storage values|1|
|[G-07]|Functions guaranteed to revert when called by normal users can be marked payable|13|
|[G-08]|Use constants instead of type(uintx).max|7|
|[G-09]|Use Assembly To Check For address(0)|1|
|[G-10]|Can Make The Variable Outside The Loop To Save Gas|5|
|[G-11]|With assembly, .call (bool success) transfer can be done gas-optimized|3|
|[G-12]|State variables can be packed to use fewer storage slots|~|
|[G-13]|Use assembly to write address storage values|5|
|[G-14]|Amounts should be checked for 0 before calling a transfer|12|
|[G-15]|internal functions not called by the contract should be removed to save deployment gas|6|
|[G-16]|Using Private Rather Than Public For Constants, Saves Gas|3|
|[G-17]|Using bools for storage incurs overhead|2|

## [G-01] Use calldata instead of memory for function arguments that do not get mutated
Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol
159 function buyOptions(
    uint poolId, 
    address[] memory options, 
    uint[] memory amounts, 
    address[] memory sourceSwap
  )
    external
  {

189 function liquidate (
    uint poolId, 
    address user,
    address[] memory options, 
    uint[] memory amounts,
    address collateralAsset
  )
    external
  {
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L159-L166

## [G‑02] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
File:  contracts/helper/LPOracle.sol
33 		decimalsA = IERC20(UniswapV2Pair(lpToken).token0()).decimals();

34		decimalsB = IERC20(UniswapV2Pair(lpToken).token1()).decimals();
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L33-L34

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol
135   (uint256 amount0, uint256 amount1) = TokenisableRange(flashAsset).withdraw(flashAmount, 0, 0);

268   TokenisableRange(debtAsset).claimFee();

289   amtA = ERC20(token0).balanceOf(user);

290   amtB = ERC20(token1).balanceOf(user);

312   uint amt0 = ERC20(token0).balanceOf(address(this));

313   uint amt1 = ERC20(token1).balanceOf(address(this));
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L135

```solidity
File:  contracts/GeVault.sol
83   (address lpap, address _token0, address _token1,, ) = RoeRouter(roeRouter).pools(poolId);

168      (ERC20 t0,) = TokenisableRange(tr).TOKEN0();

169      (ERC20 t1,) = TokenisableRange(tr).TOKEN1();
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L83

## [G-03] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops)


```solidity
File:  contracts/GeVault.sol
154    for (uint k = 0; k < ticks.length - 2; k++) 

431    for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L154

## [G-04] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)


```solidity
File:  contracts/helper/V3Proxy.sol
115          ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
116          ogInAsset.safeApprove(address(ROUTER), amountIn);


127           ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);
128           ogInAsset.safeApprove(address(ROUTER), amountInMax);


131          ogInAsset.safeTransfer(msg.sender, ogInAsset.balanceOf(address(this)));
132          ogInAsset.safeApprove(address(ROUTER), 0);


164          ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);
165          ogInAsset.safeApprove(address(ROUTER), amountInMax);

182           ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
183           ogInAsset.safeApprove(address(ROUTER), amountIn);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L115-116

```solidity
File:  contracts/PositionManager/PositionManager.sol
139      uint decimalsA = ERC20(assetA).decimals();
140      uint decimalsB = ERC20(assetB).decimals();
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L139-L140

```solidity
File:  contracts/GeVault.sol
139       (ERC20 t0,) = t.TOKEN0();
140       (ERC20 t1,) = t.TOKEN1();


168      (ERC20 t0,) = TokenisableRange(tr).TOKEN0();
169      (ERC20 t1,) = TokenisableRange(tr).TOKEN1();
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L139-L140

## [G-05] Use assembly to emit an event
To efficiently emit events, it's possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion.

However, it's important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

A good example of such practice can be seen in [Solady's](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC1155.sol#L167) codebase.

[Reffrence](https://gist.github.com/CloudEllie/05fda0fdecba2ed7d85f68c709e46c8d#gas-17-use-assembly-to-emit-an-event)

```solidity
File:  contracts/GeVault.sol
103    emit SetEnabled(_isEnabled);

110    emit SetTreasury(newTreasury);

131    emit PushTick(tr);

160    emit ShiftTick(tr);

172    emit ModifyTick(tr, index);

186   emit SetFee(newBaseFeeX4);

193    emit SetTvlCap(newTvlCap);

240    emit Withdraw(msg.sender, token, amount, liquidity);

283   emit Deposit(msg.sender, token, amount, liquidity);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L103

```solidity
File:  contracts/RangeManager.sol
87  emit AddRange(startX10, endX10, tokenisedRanges.length - 1);

120   emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);

132  emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);

164  emit Deposit(msg.sender, address(tr), lpAmt);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L87

```solidity
File:  contracts/RoeRouter.sol
50    emit DeprecatePool(poolId);

78    emit AddPool(poolId, lendingPoolAddressProvider);

86     emit UpdateTreasury(newTreasury);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L50

## [G-06] Avoid emitting storage values
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

```solidity
File:  contracts/GeVault.sol
362   emit Rebalance(tickIndex);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362


## [G-07] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
File:  contracts/GeVault.sol
101   function setEnabled(bool _isEnabled) public onlyOwner { 

108   function setTreasury(address newTreasury) public onlyOwner { 

116    function pushTick(address tr) public onlyOwner {

137     function shiftTick(address tr) public onlyOwner {

167   function modifyTick(address tr, uint index) public onlyOwner {

183   function setBaseFee(uint newBaseFeeX4) public onlyOwner {

191   function setTvlCap(uint newTvlCap) public onlyOwner {                        
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101

```solidity
File:  contracts/RangeManager.sol
75   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

95    function initRange(address tr, uint amount0, uint amount1) external onlyOwner {    
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75

```solidity
File:  contracts/RoeRouter.sol
48   function deprecatePool(uint poolId) public onlyOwner {

59     function addPool(
    address lendingPoolAddressProvider, 
    address token0, 
    address token1, 
    address ammRouter
  ) 
    public onlyOwner 
    returns (uint poolId)
  {

83  function setTreasury(address newTreasury) public onlyOwner {
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L48

```solidity
File:  contracts/helper/FixedOracle.sol
24   function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24



## [G-08] Use constants instead of type(uintx).max
 it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:
```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;
    
    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");
        
        // Do something
    }
}
```
In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```solidity
File:  contracts/GeVault.sol
386   if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L386

```solidity
File:  contracts/RangeManager.sol
118   trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));

130  uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L118

```solidity
File:  contracts/TokenisableRange.sol
172  amount0Max: type(uint128).max,

173  amount1Max: type(uint128).max
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L172

```solidity
File:  contracts/helper/OracleConvert.sol
79  return (type(uint80).max, latestAnswer(), block.timestamp, block.timestamp, type(uint80).max);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L79

```solidity
File:  contracts/PositionManager/PositionManager.sol
81    if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L81

## [G-09] Use Assembly To Check For address(0)
it's generally more gas-efficient to use assembly to check for a zero address (address(0)) than to use the == operator.

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.

Here's an example of how you can use assembly to check for a zero address:

```
contract MyContract {
    function isZeroAddress(address addr) public pure returns (bool) {
        uint256 addrInt = uint256(addr);
        
        assembly {
            // Load the zero address into memory
            let zero := mload(0x00)
            
            // Compare the address to the zero address
            let isZero := eq(addrInt, zero)
            
            // Return the result
            mstore(0x00, isZero)
            return(0, 0x20)
        }
    }
}
```
In the above example, we have a function isZeroAddress that takes an address as input and returns a boolean value indicating whether the address is equal to the zero address. Inside the function, we convert the address to an integer using uint256(addr), and then use assembly to compare the integer to the zero address.

By using assembly to perform the zero address check, we can make our code more gas-efficient and reduce the overall cost of our contract. It's important to note that assembly can be more difficult to read and maintain than Solidity code, so it should be used with caution and only when necessary

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol
136   if (sourceSwap != address(0) ){
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L136

## [G-10] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

```solidity
File:  contracts/GeVault.sol
301   address aTick = lendingPool.getReserveData(address(t)).aTokenAddress;

302   uint bal = ERC20(aTick).balanceOf(address(this));

303   (uint amt0, uint amt1) = t.getTokenAmounts(bal);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L301-L303

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol
72    address asset = assets[k];

73    uint amount = amounts[k];
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L72-L73

## [G-11] With assembly, .call (bool success) transfer can be done gas-optimized
return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity
File:  contracts/helper/V3Proxy.sol
156   msg.sender.call{value: msg.value - amounts[0]}("");

174   payable(msg.sender).call{value: amountOut}("");

192   payable(msg.sender).call{value: amounts[1]}("");
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156

## [G-12] State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

```solidity
File:  contracts/TokenisableRange.sol
  int24 public lowerTick;
  int24 public upperTick;
  uint24 public feeTier;
  
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
  
  uint128 public liquidity;
  // @notice deprecated, keep to avoid beacon storage slot overwriting errors
  address public TREASURY_DEPRECATED = 0x22Cc3f665ba4C898226353B672c5123c58751692;
  uint public treasuryFee_deprecated = 20;
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L28-L55

## [G-13] Use assembly to write address storage values
By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```

```solidity
File:  contracts/GeVault.sol
89   treasury = _treasury;

90   uniswapPool = IUniswapV3Pool(_uniswapPool);

91    WETH = IWETH(weth);

109   treasury = newTreasury; 
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L89

```solidity
File:  contracts/RoeRouter.sol
35   treasury = treasury_;
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L35

## [G-14] Amounts should be checked for 0 before calling a transfer
It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```
In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
File:  contracts/GeVault.sol
227   ERC20(token).safeTransfer(treasury, fee);

232   payable(msg.sender).transfer(bal);

235   ERC20(token).safeTransfer(msg.sender, bal);

267   ERC20(token).safeTransfer(treasury, fee);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L227

```solidity
File:  contracts/RangeManager.sol
96   ASSET_0.safeTransferFrom(msg.sender, address(this), amount0);

98   ASSET_1.safeTransferFrom(msg.sender, address(this), amount1);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L96

```solidity
File:  contracts/TokenisableRange.sol
138      TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);

139      TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138

```solidity
File:  contracts/helper/V3Proxy.sol
115  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);

164  ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);

182  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L115

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol
370   IERC20(collateralAsset).safeTransfer(ROEROUTER.treasury(), feeAmount);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L370


## [G-15] internal functions not called by the contract should be removed to save deployment gas
When you define an internal function in a Solidity contract, Solidity generates code for that function and includes it in the contract bytecode. If the function is not called by the contract itself or any of its external functions, then 

```solidity
File:  contracts/PositionManager/PositionManager.sol
63    function getPoolAddresses(uint poolId) 
    internal view 
    returns( ILendingPool lp, IPriceOracle oracle, IUniswapV2Router01 router, address token0, address token1) 
  {

89   function cleanup(ILendingPool LP, address user, address asset) internal {

124  function PMWithdraw(ILendingPool LP, address user, address asset, uint amount) internal {

138  function validateValuesAgainstOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB, uint amountB) internal view {            
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L63

```solidity
File:  interfaces/PoolAddress.sol
20      function getPoolKey(
        address tokenA,
        address tokenB,
        uint24 fee
    ) internal pure returns (PoolKey memory) {

33   function computeAddress(address factory, PoolKey memory key) internal pure returns (address pool) {
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L20

## [G‑16] Using Private Rather Than Public For Constants, Saves Gas
When you declare a constant variable as public, Solidity generates a getter function that allows anyone to read the value of the constant. This getter function can consume gas, especially if the constant is read frequently or the contract is called by multiple users.

By using private instead of public for constants, you can prevent the generation of the getter function and reduce the overall gas cost of your contract.

```solidity
File:  contracts/GeVault.sol
62   uint public constant nearbyRanges = 2;
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L62

```solidity
File:  contracts/TokenisableRange.sol
60   address constant public treasury = 0x22Cc3f665ba4C898226353B672c5123c58751692;

61   uint constant public treasuryFee = 20;
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L60

## [G-17] Using bools for storage incurs overhead
```  
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.

    Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from 'false' to 'true', after having been 'true' in the past
```

### Note: missed from bots

```solidity
File:  contracts/GeVault.sol
64   bool public baseTokenIsToken0;
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L64

```solidity
File:  contracts/helper/V3Proxy.sol
65   bool acceptPayable;
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L65
