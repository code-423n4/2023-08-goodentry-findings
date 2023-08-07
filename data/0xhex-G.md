# Gas Optimization

# Summary

| Number | Optimization Gas Details                                                               | Instances |
| :----: | :------------------------------------------------------------------------------------- | :-------: |
| [G-01] | Use assembly to emit an event                                                          |    12     |
| [G-02] | Avoid contract existence checks by using low level calls                               |     5     |
| [G-03] | With assembly, .call (bool success) transfer can be done gas-optimized                 |     3     |
| [G-04] | Functions guaranteed to when called by normal users can be marked Payable              |    13     |
| [G-05] | State varaibles only set in the Constructor should be declared Immutable               |    14     |
| [G-06] | Use constants instead of type(uintx).max                                               |     6     |
| [G-07] | Use calldata instead of memory for function arguments that do not get mutated          |     5     |
| [G-08] | Use assembly to write address storage values                                           |     5     |
| [G-09] | Amounts should be checked for 0 before calling a transfer                              |    12     |
| [G-10] | Using PRIVATE rather than PUBLIC FOR Constants, Saves Gas                              |     6     |
| [G-11] | Not using the named return variable when a function returns, wastes deployment gas     |     5     |
| [G-12] | Can Make The Variable Outside The Loop To Save Gas                                     |     6     |
| [G-13] | Duplicated require()/if() checks should be refactored to a modifier or function        |    10     |
| [G-14] | Use Assembly To Check For address(0)                                                   |     1     |
| [G-15] | Internal functions not called by the contract should be removed to save deployment gas |     6     |
| [G-16] | Should use arguments instead of state variable                                         |     1     |
| [G-17] | Empty blocks should be removed or emit something                                       |     2     |
| [G-18] | Use hardcode address instead address(this)                                             |     6     |
| [G-19] | Sort Solidity operations using short-circuit mode                                      |     2     |
| [G-20] | Require() statements that check input arguments should be at the top of the function   |     6     |
| [G-21] | Using delete statement can save gas                                                    |     1     |
| [G-22] | abi.encode() is less efficient than abi.encodepacked()                                 |     2     |

## [G-01] Use assembly to emit an event

To efficiently emit events, it's possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion.

However, it's important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

A Good example of such practice can be seen in [Solady's](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC1155.sol#L167) codebase.

```solidity
File:  contracts/RangeManager.sol

87  emit AddRange(startX10, endX10, tokenisedRanges.length - 1);

120   emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);

132  emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);

164  emit Deposit(msg.sender, address(tr), lpAmt);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L87

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

## [G-02] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
file: /contracts/GeVault.sol

87    lendingPool = ILendingPool(ILendingPoolAddressesProvider(lpap).getLendingPool());

88    oracle = IPriceOracle(ILendingPoolAddressesProvider(lpap).getPriceOracle());

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L87-L88

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

135    (uint256 amount0, uint256 amount1) = TokenisableRange(flashAsset).withdraw(flashAmount, 0, 0);

270      (uint token0Amount, uint token1Amount) = TokenisableRange(debtAsset).getTokenAmounts(debt);

305      debt = TokenisableRange(debtAsset).deposit(token0Amount, token1Amount);

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L135

## [G-03] With assembly, .call (bool success) transfer can be done gas-optimized

Return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity
File:  contracts/helper/V3Proxy.sol

156   msg.sender.call{value: msg.value - amounts[0]}("");

174   payable(msg.sender).call{value: amountOut}("");

192   payable(msg.sender).call{value: amounts[1]}("");
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156

## [G-04] Functions guaranteed to when called by normal users can be marked Payable

```solidity
file: /contracts/RangeManager.sol

75     function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

95     function initRange(address tr, uint amount0, uint amount1) external onlyOwner {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75

```solidity
file: /contracts/helper/V3Proxy.sol

94     function emergencyWithdraw(ERC20 token) onlyOwner external {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L94

```solidity
file: /contracts/RoeRouter.sol

48    function deprecatePool(uint poolId) public onlyOwner {

65    public onlyOwner

83    function setTreasury(address newTreasury) public onlyOwner {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L48

```solidity
file: /contracts/GeVault.sol

101    function setEnabled(bool _isEnabled) public onlyOwner {

108    function setTreasury(address newTreasury) public onlyOwner {

116    function pushTick(address tr) public onlyOwner {

137    function shiftTick(address tr) public onlyOwner {

167    function modifyTick(address tr, uint index) public onlyOwner {

183    function setBaseFee(uint newBaseFeeX4) public onlyOwner {

191    function setTvlCap(uint newTvlCap) public onlyOwner {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101

## [G-05] State varaibles only set in the Constructor should be declared Immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

```solidity
file: /contracts/RangeManager.sol

50    LENDING_POOL = lendingPool;

51    ASSET_0 = _asset0 < _asset1 ? _asset0 : _asset1;

52    ASSET_1 = _asset0 < _asset1 ? _asset1 : _asset0;

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L50-L52

```solidity
file: /contracts/GeVault.sol

84    token0 = ERC20(_token0);

85    token1 = ERC20(_token1);

87    lendingPool = ILendingPool(ILendingPoolAddressesProvider(lpap).getLendingPool());

88    oracle = IPriceOracle(ILendingPoolAddressesProvider(lpap).getPriceOracle());

89    treasury = _treasury;

90    uniswapPool = IUniswapV3Pool(_uniswapPool);

91    WETH = IWETH(weth);

92    baseTokenIsToken0 = _baseTokenIsToken0;

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L84-L92

```solidity
file: /contracts/RoeRouter.sol

35    treasury = treasury_;

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L35

```solidity
file: /contracts/helper/FixedOracle.sol

16        owner = msg.sender;

17        hardcodedPrice = _hardcodedPrice;

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L16-L17

## [G-06] Use constants instead of type(uintx).max

It's generally more gas-efficient to use constants instead of `type(uintX).max` when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

```solidity
File:  contracts/TokenisableRange.sol

172  amount0Max: type(uint128).max,

173  amount1Max: type(uint128).max
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L172

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
File:  contracts/helper/OracleConvert.sol

79  return (type(uint80).max, latestAnswer(), block.timestamp, block.timestamp, type(uint80).max);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L79

## [G-07] Use calldata instead of memory for function arguments that do not get mutated

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

159  function buyOptions(
      uint poolId,
      address[] memory options,
      uint[] memory amounts,
      address[] memory sourceSwap
      )
165    external

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L159-L165

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

```solidity
file: /contracts/TokenisableRange.sol

85     function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L85

```solidity
file: /contracts/RangeManager.sol

75    function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75

## [G-08] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

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

## [G-09] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

```solidity
File:  contracts/TokenisableRange.sol

138      TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);

139      TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138

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

## [G-10] Using PRIVATE rather than PUBLIC FOR Constants, Saves Gas

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```solidity
file: /contracts/GeVault.sol

62  uint public constant nearbyRanges = 2;

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L62

```solidity
file: /contracts/TokenisableRange.sol

58     INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);

59     IUniswapV3Factory constant public V3_FACTORY = IUniswapV3Factory(0x1F98431c8aD98523631AE4a59f267346ea31F984);

60     address constant public treasury = 0x22Cc3f665ba4C898226353B672c5123c58751692;

61     uint constant public treasuryFee = 20;

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L58-L61

```solidity
file: /contracts/RangeManager.sol

36       INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36

## [G-11] Not using the named return variable when a function returns, wastes deployment gas

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

514    internal view returns (uint amountB)

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L514

```solidity
file: /contracts/GeVault.sol

177  function getTickLength() public view returns(uint len){

298  function getReserves() public view returns (uint amount0, uint amount1){

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L177

```solidity
file: /contracts/RangeManager.sol

212  function getStepListLength() external view returns (uint256 listLength) {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L212

## [G-12] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol

72    address asset = assets[k];

73    uint amount = amounts[k];
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L72-L73

```solidity
File:  contracts/GeVault.sol

301   address aTick = lendingPool.getReserveData(address(t)).aTokenAddress;

302   uint bal = ERC20(aTick).balanceOf(address(this));

303   (uint amt0, uint amt1) = t.getTokenAmounts(bal);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L301-L303

## [G-13] Duplicated require()/if() checks should be refactored to a modifier or function

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol


69        require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");

91        require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");



393    require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

420    require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L69

```solidity
file: /contracts/GeVault.sol


120    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
141    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
170    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");



203    require(poolMatchesOracle(), "GEV: Oracle Error");
215    require(poolMatchesOracle(), "GEV: Oracle Error");
250    require(poolMatchesOracle(), "GEV: Oracle Error");
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120

## [G-14] Use Assembly To Check For address(0)

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol

136   if (sourceSwap != address(0) ){
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L136

## [G-15] Internal functions not called by the contract should be removed to save deployment gas

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

## [G-16] Should use arguments instead of state variable

state variables should not used in emit , This will save near 97 gas

```solidity
file: /contracts/GeVault.sol

362    emit Rebalance(tickIndex);

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362

## [G-17] Empty blocks should be removed or emit something

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

25      constructor (address roerouter) PositionManager(roerouter) {}

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L25

```solidity
file: /contracts/PositionManager/PositionManager.sol

52       ) virtual external returns (bool result) {
53          }

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L52-L53

## [G-18] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.(References)[https://book.getfoundry.sh/reference/forge-std/compute-create-address]

```solidity
file: /contracts/GeVault.sol

386      if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

409    uint bal = t.balanceOf(address(this));

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L386

```solidity
file: /contracts/TokenisableRange.sol

138    TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);

139    TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138-L139

```solidity
file: /contracts/RangeManager.sol

118        trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L118

```solidity
file: /contracts/helper/V3Proxy.sol

167        amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountOut, amountInMax, 0));

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L167

## [G-19] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation
//g(y) is a high gas cost operation
//Sort operations with different gas costs as follows
 f(x) || g(y)
 f(x) && g(y)
```

```solidity
file: /contracts/RangeManager.sol

63      if (start >= stepList[i].end || end <= stepList[i].start) {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L63

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

495    require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L495

## [G-20] Require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas\*) in a function that may ultimately revert in the unhappy case.

```solidity
file: /contracts/TokenisableRange.sol

209      require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L209

```solidity
file: /contracts/PositionManager/PositionManager.sol

145    require( valueA <= valueB * 101 / 100, "PM: LP Oracle Error");

146    require( valueB <= valueA * 101 / 100, "PM: LP Oracle Error");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L145-L146

```soldity
file: /contracts/helper/LPOracle.sol

101    require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L101

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

369    require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L369

```solidity
file: /contracts/RangeManager.sol

206    require(hf > 1.01e18, "Health factor is too low");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L206

## [G-21] Using delete statement can save gas

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

204      flashtype[i] = 0; // dont open debt for liquidations, need to repay

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L204

## [G-22] abi.encode() is less efficient than abi.encodepacked()

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

168    bytes memory params = abi.encode(0, poolId, msg.sender, sourceSwap);

199    bytes memory params = abi.encode(1, poolId, user, collateralAsset); // mode = 1 -> liquidation

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L168