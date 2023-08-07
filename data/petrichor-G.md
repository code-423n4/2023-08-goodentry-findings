# Gas Optimization 

# summary

|      |  issue  | instance |
|------|---------|----------|
|[G-01]|Using Private Rather Than Public For Constants, Saves Gas|3|
|[G-02]| internal functions not called by the contract should be removed to save deployment gas|6|
|[G-03]|Amounts should be checked for 0 before calling a transfer|12|
|[G-04]|Use do while loops instead of for loops|2|
|[G-05]|Functions guaranteed to revert when called by normal users can be marked payable|12|
|[G-06]|Avoid contract existence checks by using low level calls|11|
|[G-07]|Use calldata instead of memory for function arguments that do not get mutated|2|
|[G-08]|Can Make The Variable Outside The Loop To Save Gas |5|
|[G-09]|Use assembly to write address storage values|5|
|[G-10]|State variables can be packed to use fewer storage slots|1|
|[G-11]| Use constants instead of type(uintx).max|7|
|[G-12]|Avoid emitting storage values|1|
|[G-13]|Use Assembly To Check For address(0)|1|
|[G-14]|Use assembly to emit an event|16|
|[G-15]|With assembly, .call (bool success) transfer can be done gas-optimized|3|




## [G‑01] Using Private Rather Than Public For Constants, Saves Gas



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
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L61


## [G-02] internal functions not called by the contract should be removed to save deployment gas

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
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L89
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L124
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L138


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
https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L33


## [G-03] Amounts should be checked for 0 before calling a transfer


Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.


```solidity
File:  contracts/GeVault.sol
227   ERC20(token).safeTransfer(treasury, fee);

232   payable(msg.sender).transfer(bal);

235   ERC20(token).safeTransfer(msg.sender, bal);

267   ERC20(token).safeTransfer(treasury, fee);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L227
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L232
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L235
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L267


```solidity
File:  contracts/RangeManager.sol
96   ASSET_0.safeTransferFrom(msg.sender, address(this), amount0);

98   ASSET_1.safeTransferFrom(msg.sender, address(this), amount1);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L96
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L98


```solidity
File:  contracts/TokenisableRange.sol
138      TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);

139      TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L139


```solidity
File:  contracts/helper/V3Proxy.sol
115  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);

164  ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);

182  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L115
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L164
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L182


```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol
370   IERC20(collateralAsset).safeTransfer(ROEROUTER.treasury(), feeAmount);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L370


## [G-04] Use do while loops instead of for loops


```solidity
File:  contracts/GeVault.sol
154    for (uint k = 0; k < ticks.length - 2; k++) 

431    for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L154
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L431


## [G-05] Functions guaranteed to revert when called by normal users can be marked payable


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
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L108
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L116
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L137
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L167
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L183
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L191


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
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L59
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L83


```solidity
File:  contracts/helper/FixedOracle.sol
24   function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24


## [G‑06] Avoid contract existence checks by using low level calls



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
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L268
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L289
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L290
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L312
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L313
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L135


```solidity
File:  contracts/GeVault.sol
83   (address lpap, address _token0, address _token1,, ) = RoeRouter(roeRouter).pools(poolId);

168      (ERC20 t0,) = TokenisableRange(tr).TOKEN0();

169      (ERC20 t1,) = TokenisableRange(tr).TOKEN1();
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L83
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L168
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L169




## [G-07] Use calldata instead of memory for function arguments that do not get mutated

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
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L159-L159

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L159-L189



## [G-08] Can Make The Variable Outside The Loop To Save Gas 

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. 


```solidity
File:  contracts/GeVault.sol
301   address aTick = lendingPool.getReserveData(address(t)).aTokenAddress;
302   uint bal = ERC20(aTick).balanceOf(address(this));
303   (uint amt0, uint amt1) = t.getTokenAmounts(bal);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L301-
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L302
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L303


```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol
72    address asset = assets[k];
73    uint amount = amounts[k];
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L72
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L73


## [G-09] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.



```solidity
File:  contracts/GeVault.sol
89   treasury = _treasury;

90   uniswapPool = IUniswapV3Pool(_uniswapPool);

91    WETH = IWETH(weth);

109   treasury = newTreasury; 
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L89
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L90
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L91
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L109



```solidity
File:  contracts/RoeRouter.sol
35   treasury = treasury_;
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L35




## [G-10] State variables can be packed to use fewer storage slot
To optimize gas usage by reducing the number of storage slots used in the EVM (Ethereum Virtual Machine), you can consider combining related variables into a single struct. This allows you to store multiple variables within a single storage slot.


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

### replace code

pragma solidity ^0.8.0;

contract TokenisableRange {
    struct ASSET {
        ERC20 token;
        uint8 decimals;
    }
    
    struct TokenisableRangeData {
        int24 lowerTick;
        int24 upperTick;
        uint24 feeTier;
        uint256 tokenId;
        uint256 fee0;
        uint256 fee1;
        ASSET token0;
        ASSET token1;
        IAaveOracle oracle;
        string name;
        string symbol;
        ProxyState status;
        address creator;
        uint128 liquidity;
    }
    
    TokenisableRangeData public data;
    
    enum ProxyState { INIT_PROXY, INIT_LP, READY }

    constructor(
        int24 _lowerTick,
        int24 _upperTick,
        uint24 _feeTier,
        uint256 _tokenId,
        uint256 _fee0,
        uint256 _fee1,
        ERC20 _token0,
        ERC20 _token1,
        IAaveOracle _oracle,
        string memory _name,
        string memory _symbol,
        ProxyState _status,
        address _creator,
        uint128 _liquidity
    ) {
        data.lowerTick = _lowerTick;
        data.upperTick = _upperTick;
        data.feeTier = _feeTier;
        data.tokenId = _tokenId;
        data.fee0 = _fee0;
        data.fee1 = _fee1;
        data.token0.token = _token0;
        data.token0.decimals = _token0.decimals();
        data.token1.token = _token1;
        data.token1.decimals = _token1.decimals();
        data.oracle = _oracle;
        data.name = _name;
        data.symbol = _symbol;
        data.status = _status;
        data.creator = _creator;
        data.liquidity = _liquidity;
    }
}

### In the refactored code, the related variables have been grouped together into a single TokenisableRangeData struct. By doing this, you can store the combined struct in a single storage slot, resulting in gas savings.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L28-L55



## [G-11] Use constants instead of type(uintx).max
 

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.
By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.


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
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L130


```solidity
File:  contracts/TokenisableRange.sol
172  amount0Max: type(uint128).max,

173  amount1Max: type(uint128).max
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L172
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L173


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


## [G-12]Avoid emitting storage values

```solidity
File:  contracts/GeVault.sol
362   emit Rebalance(tickIndex);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362




## [G-13] Use Assembly To Check For address(0)

The reason for this is that the == operator generates additional instructions in the EVM bytecode, which can increase the gas cost of your contract. By using assembly, you can perform the zero address check more efficiently and reduce the overall gas cost of your contract.


```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol
136   if (sourceSwap != address(0) ){
```


https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L136


## [G-14] Use assembly to emit an event




To efficiently emit events, it's possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion.

However, it's important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

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
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L110
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L131
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L160
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L172
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L186
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L193
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L240
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L283


```solidity
File:  contracts/RangeManager.sol
87  emit AddRange(startX10, endX10, tokenisedRanges.length - 1);

120   emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);

132  emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);

164  emit Deposit(msg.sender, address(tr), lpAmt);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L87
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L120
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L132
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L164


```solidity
File:  contracts/RoeRouter.sol
50    emit DeprecatePool(poolId);

78    emit AddPool(poolId, lendingPoolAddressProvider);

86     emit UpdateTreasury(newTreasury);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L50
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L78
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L86




## [G-15] With assembly, .call (bool success) transfer can be done gas-optimized
Performing a gas-optimized transfer using assembly's .call function can potentially save gas compared to using the transfer function or directly assigning values to the address type

```solidity
File:  contracts/helper/V3Proxy.sol
156   msg.sender.call{value: msg.value - amounts[0]}("");

174   payable(msg.sender).call{value: amountOut}("");

192   payable(msg.sender).call{value: amounts[1]}("");
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L174
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L192

