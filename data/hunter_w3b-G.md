# Gas Optimization

# Summary

| Number | Optimization Details                                                                                             | Context |
| :----: | :--------------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Functions guaranteed to when called by normal users can be marked `Payable`                                      |   14    |
| [G-02] | State varaibles only set in the Constructor should be declared `Immutable`                                       |   16    |
| [G-03] | Use hardcode address instead `address(this)`                                                                     |    7    |
| [G-04] | Use constants instead of `type(uintx).max`                                                                       |    7    |
| [G-05] | Use assembly to `emit` an event                                                                                  |   17    |
| [G-06] | Sort Solidity operations using `short-circuit` mode                                                              |    2    |
| [G-07] | Avoid contract existence checks by using low level calls                                                         |    6    |
| [G-08] | Amounts should be checked for 0 before calling a transfer                                                        |   12    |
| [G-09] | Using delete statement can save gas                                                                              |    1    |
| [G-10] | Using `PRIVATE` rather than `PUBLIC` FOR Constants, Saves Gas                                                    |    6    |
| [G-11] | Not using the named return variable when a function returns, wastes deployment gas                               |    4    |
| [G-12] | Use calldata instead of memory for function arguments that do not get mutated                                    |   12    |
| [G-13] | Can Make The Variable Outside The Loop To Save Gas                                                               |    5    |
| [G-14] | Internal functions not called by the contract should be removed to save deployment gas                           |    6    |
| [G-15] | Should use arguments instead of state variable                                                                   |    1    |
| [G-16] | Require() statements that check input arguments should be at the top of the function                             |    6    |
| [G-17] | Empty blocks should be removed or emit something                                                                 |    2    |
| [G-18] | With assembly, `.call` (bool success) transfer can be done gas-optimized                                         |    3    |
| [G-19] | Use `assembly` to write address storage values                                                                   |    5    |
| [G-20] | Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it) |    1    |
| [G-21] | Duplicated require()/if() checks should be refactored to a modifier or function                                  |    5    |
| [G-22] | Use Assembly To Check For `address(0)`                                                                           |    1    |
| [G-23] | `abi.encode()` is less efficient than `abi.encodepacked()`                                                       |    2    |
| [G-24] | Require() string longer than 32 bytes cost extra gas                                                             |    1    |
| [G-25] | Splitting `require()` statements that use && saves gas                                                           |    7    |
| [G-26] | State variables can be packed to use fewer storage slots                                                         |    1    |

## [G-01] Functions guaranteed to when called by normal users can be marked Payable

The `onlyOwner` modifier makes a function revert if not called by the address registered as the owner

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101

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

```diff

-101    function setEnabled(bool _isEnabled) public onlyOwner {
+101    function setEnabled(bool _isEnabled) public payable onlyOwner {


-108    function setTreasury(address newTreasury) public onlyOwner {
+108    function setTreasury(address newTreasury) public payable onlyOwner {

-116    function pushTick(address tr) public onlyOwner {
+116    function pushTick(address tr) public payable onlyOwner {

-137    function shiftTick(address tr) public onlyOwner {
+137    function shiftTick(address tr) public payable onlyOwner {

-167    function modifyTick(address tr, uint index) public onlyOwner {
+167    function modifyTick(address tr, uint index) public payable onlyOwner {


```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L94

```solidity
file: /contracts/helper/V3Proxy.sol

94     function emergencyWithdraw(ERC20 token) onlyOwner external {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75

```solidity
file: /contracts/RangeManager.sol

75     function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

95     function initRange(address tr, uint amount0, uint amount1) external onlyOwner {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L48

```solidity
file: /contracts/RoeRouter.sol

48    function deprecatePool(uint poolId) public onlyOwner {

65    public onlyOwner

83    function setTreasury(address newTreasury) public onlyOwner {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24

```solidity
file: /contracts/helper/FixedOracle.sol

24    function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {

```

## [G-02] State varaibles only set in the Constructor should be declared `Immutable`

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

`Total gas = 16 * 20000 = 320000`

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L50-L52

```solidity
file: /contracts/RangeManager.sol

50    LENDING_POOL = lendingPool;

51    ASSET_0 = _asset0 < _asset1 ? _asset0 : _asset1;

52    ASSET_1 = _asset0 < _asset1 ? _asset1 : _asset0;

```

```diff
diff --git a/RangeManager.org.sol b/RangeManager.sol
index c7254c7..3a4bea5 100644
--- a/RangeManager.org.sol
+++ b/RangeManager.sol
@@ -1,11 +1,11 @@
   using SafeERC20 for ERC20;
-  ILendingPool public LENDING_POOL;
+  ILendingPool public immutable LENDING_POOL;
   event Withdraw(address user, address asset, uint amount);
   event Deposit(address user, address asset, uint amount);
   event AddRange(uint128 startX10, uint128 endX10, uint step);

-  ERC20 public ASSET_0;
-  ERC20 public ASSET_1;
+  ERC20 public immutable ASSET_0;
+  ERC20 public immutable ASSET_1;

   // Constant across chains - https://docs.uniswap.org/protocol/reference/deployments
   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L31

```solidity
file: /contracts/PositionManager/PositionManager.sol

31    ROEROUTER = RoeRouter(roerouter);

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L84-L92

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L35

```solidity
file: /contracts/RoeRouter.sol

35    treasury = treasury_;

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L16-L17

```solidity
file: /contracts/helper/FixedOracle.sol

16        owner = msg.sender;

17        hardcodedPrice = _hardcodedPrice;

```

## [G-03] Use hardcode address instead `address(this)`

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.(References)[https://book.getfoundry.sh/reference/forge-std/compute-create-address]

If the contract's address is needed in the code, it's more gas-efficient to hardcode the address as a constant rather than using the address(this) expression. This is because using address(this) requires additional gas consumption to compute the contract's address at runtime.

Here's an example :

```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;

    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L386

```solidity
file: /contracts/GeVault.sol

386      if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

409    uint bal = t.balanceOf(address(this));

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138-L139

```solidity
file: /contracts/TokenisableRange.sol

138    TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);

139    TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L167

```solidity
file: /contracts/helper/V3Proxy.sol

167        amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountOut, amountInMax, 0));

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L118

```solidity
file: /contracts/RangeManager.sol

118        trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));

```

## [G-04] Use constants instead of `type(uintx).max`

It's generally more gas-efficient to use constants instead of `type(uintX).max` when you need to set the maximum value of an unsigned integer type.

The reason for this is that the `type(uintX).max` expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using `type(uintX).max` can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of `type(uintX).max`, you can avoid these additional gas costs and make your code more efficient.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L81

```solidity
File:  contracts/PositionManager/PositionManager.sol

81    if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L118

```solidity
File:  contracts/RangeManager.sol

118   trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));

130  uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L172

```solidity
File:  contracts/TokenisableRange.sol

172  amount0Max: type(uint128).max,

173  amount1Max: type(uint128).max
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L79

```solidity
File:  contracts/helper/OracleConvert.sol

79  return (type(uint80).max, latestAnswer(), block.timestamp, block.timestamp, type(uint80).max);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L386

```solidity
File:  contracts/GeVault.sol

386   if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);
```

## [G-05] Use assembly to `emit` an event

To efficiently emit events, it's possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion.

However, it's important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

A Good example of such practice can be seen in [Solady's](https://github.com/Vectorized/solady/blob/main/src/tokens/ERC1155.sol#L167) codebase.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L87

```solidity
File:  contracts/RangeManager.sol

87  emit AddRange(startX10, endX10, tokenisedRanges.length - 1);

120   emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);

132  emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);

164  emit Deposit(msg.sender, address(tr), lpAmt);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L103

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L50

```solidity
File:  contracts/RoeRouter.sol

50    emit DeprecatePool(poolId);

78    emit AddPool(poolId, lendingPoolAddressProvider);

86     emit UpdateTreasury(newTreasury);
```

## [G-06] Sort Solidity operations using `short-circuit` mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation
//g(y) is a high gas cost operation
//Sort operations with different gas costs as follows
 f(x) || g(y)
 f(x) && g(y)
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L63

```solidity
file: /contracts/RangeManager.sol

63      if (start >= stepList[i].end || end <= stepList[i].start) {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L495

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

495    require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );

```

## [G-07] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L135

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

135    (uint256 amount0, uint256 amount1) = TokenisableRange(flashAsset).withdraw(flashAmount, 0, 0);

270      (uint token0Amount, uint token1Amount) = TokenisableRange(debtAsset).getTokenAmounts(debt);

305      debt = TokenisableRange(debtAsset).deposit(token0Amount, token1Amount);

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L87-L88

```solidity
file: /contracts/GeVault.sol

87    lendingPool = ILendingPool(ILendingPoolAddressesProvider(lpap).getLendingPool());

88    oracle = IPriceOracle(ILendingPoolAddressesProvider(lpap).getPriceOracle());

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L370

```solidity
file: /contracts/TokenisableRange.sol

370    (uint160 sqrtPriceX96,,,,,,)  = IUniswapV3Pool(pool).slot0();

```

## [G-08] Amounts should be checked for 0 before calling a transfer

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L115

```solidity
File:  contracts/helper/V3Proxy.sol

115  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);

164  ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);

182  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L96

```solidity
File:  contracts/RangeManager.sol

96   ASSET_0.safeTransferFrom(msg.sender, address(this), amount0);

98   ASSET_1.safeTransferFrom(msg.sender, address(this), amount1);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138

```solidity
File:  contracts/TokenisableRange.sol

138      TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);

139      TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L227

```solidity
File:  contracts/GeVault.sol

227   ERC20(token).safeTransfer(treasury, fee);

232   payable(msg.sender).transfer(bal);

235   ERC20(token).safeTransfer(msg.sender, bal);

267   ERC20(token).safeTransfer(treasury, fee);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L370

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol

370   IERC20(collateralAsset).safeTransfer(ROEROUTER.treasury(), feeAmount);
```

## [G-09] Using delete statement can save gas

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L204

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

204      flashtype[i] = 0; // dont open debt for liquidations, need to repay

```

## [G-10] Using `PRIVATE` rather than `PUBLIC` FOR Constants, Saves Gas

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L514

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

514    internal view returns (uint amountB)

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L177

```solidity
file: /contracts/GeVault.sol

177  function getTickLength() public view returns(uint len){

298  function getReserves() public view returns (uint amount0, uint amount1){

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L212

```solidity
file: /contracts/RangeManager.sol

212  function getStepListLength() external view returns (uint256 listLength) {

```

## [G-12] Use calldata instead of memory for function arguments that do not get mutated

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L159-L165

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L85

```solidity
file: /contracts/TokenisableRange.sol

85     function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L159-L166

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75

```solidity
file: /contracts/RangeManager.sol

75    function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

```

## [G-13] Can Make The Variable Outside The Loop To Save Gas

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L301-L303

```solidity
File:  contracts/GeVault.sol

301   address aTick = lendingPool.getReserveData(address(t)).aTokenAddress;
302   uint bal = ERC20(aTick).balanceOf(address(this));
303   (uint amt0, uint amt1) = t.getTokenAmounts(bal);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L72-L73

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol

72    address asset = assets[k];
73    uint amount = amounts[k];
```

## [G-14] Internal functions not called by the contract should be removed to save deployment gas

When you define an internal function in a Solidity contract, Solidity generates code for that function and includes it in the contract bytecode. If the function is not called by the contract itself or any of its external functions, then

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L63

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L20

```solidity
File:  interfaces/PoolAddress.sol


20      function getPoolKey(
        address tokenA,
        address tokenB,
        uint24 fee
    ) internal pure returns (PoolKey memory) {



33   function computeAddress(address factory, PoolKey memory key) internal pure returns (address pool) {
```

## [G-15] Should use arguments instead of state variable

state variables should not used in emit , This will save near 97 gas

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362

```solidity
file: /contracts/GeVault.sol

362    emit Rebalance(tickIndex);

```

## [G-16] Require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas\*) in a function that may ultimately revert in the unhappy case.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L101

```soldity
file: /contracts/helper/LPOracle.sol

101    require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L209

```solidity
file: /contracts/TokenisableRange.sol

209      require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L206

```solidity
file: /contracts/RangeManager.sol

206    require(hf > 1.01e18, "Health factor is too low");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L369

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

369    require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L145-L146

```solidity
file: /contracts/PositionManager/PositionManager.sol

145    require( valueA <= valueB * 101 / 100, "PM: LP Oracle Error");

146    require( valueB <= valueA * 101 / 100, "PM: LP Oracle Error");

```

## [G-17] Empty blocks should be removed or emit something

f the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) .
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L52-L53

```solidity
file: /contracts/PositionManager/PositionManager.sol

52       ) virtual external returns (bool result) {
53          }

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L25

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

25      constructor (address roerouter) PositionManager(roerouter) {}

```

## [G-18] With assembly, `.call` (bool success) transfer can be done gas-optimized

Return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156

```solidity
File:  contracts/helper/V3Proxy.sol

156   msg.sender.call{value: msg.value - amounts[0]}("");

174   payable(msg.sender).call{value: amountOut}("");

192   payable(msg.sender).call{value: amounts[1]}("");
```

## [G-19] Use assembly to write address storage values

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L89

```solidity
File:  contracts/GeVault.sol

89   treasury = _treasury;

90   uniswapPool = IUniswapV3Pool(_uniswapPool);

91    WETH = IWETH(weth);

109   treasury = newTreasury;
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L35

```solidity
File:  contracts/RoeRouter.sol

35   treasury = treasury_;
```

## [G-20] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L88

```solidity
file: /contracts/TokenisableRange.sol

88    creator = msg.sender;

```

## [G-21] Duplicated require()/if() checks should be refactored to a modifier or function

to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L69

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

69        require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");
91        require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");


393    require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );
420    require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120

```solidity
file: /contracts/GeVault.sol

120     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
141     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
170     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");


203    require(poolMatchesOracle(), "GEV: Oracle Error");
215    require(poolMatchesOracle(), "GEV: Oracle Error");
250    require(poolMatchesOracle(), "GEV: Oracle Error");
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L121

```solidity
file: /contracts/GeVault.sol

121     if (ticks.length == 0) ticks.push(t);
142     if (ticks.length == 0) ticks.push(t);

```

## [G-22] Use Assembly To Check For address(0)

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L136

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol

136   if (sourceSwap != address(0) ){
```

## [G-23] abi.encode() is less efficient than abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data. (Refference)[https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison]

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L168

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

168    bytes memory params = abi.encode(0, poolId, msg.sender, sourceSwap);

199    bytes memory params = abi.encode(1, poolId, user, collateralAsset); // mode = 1 -> liquidation

```

## [G-24] Require() string longer than 32 bytes cost extra gas

when these strings get longer than 32 bytes, they cost more gas
Use shorter require()/revert() strings

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11

```solidity
file: /contracts/helper/FixedOracle.sol

11        require(msg.sender == owner, "Only the owner can call this function.");

```

## [G-25] Splitting `require()` statements that use && saves gas

Instead of using operator && on a single require. Using a two require can save more gas.
i.e. for require(version == 1 && bytecodeHash[1] == bytes1(0) , “zf”) ; use:
require(version == 1); require(`_bytecodeHash[1]` == bytes(0));

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L68-L74

```soldity
file: /contracts/RoeRouter.sol

68    require (
      lendingPoolAddressProvider != address(0x0)
      && token0 != address(0x0)
      && token1 != address(0x0)
      && ammRouter != address(0x0),
      "Invalid Address"
74    );

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L29

```solidity
file: /contracts/helper/LPOracle.sol

29    require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L167

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

167    require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");

344    require(
       (debtValue < 1e8 && tokensValue < 1e8 )
       || (tokensValue > debtValue * 98 / 100 && tokensValue < debtValue * 102 / 100),
       "OPM: Slippage Error"
348     );

495    require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L141

```solidity
file: /contracts/GeVault.sol

141    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L108

```solidity
file: /contracts/RangeManager.sol

108    require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");

```

## [G-26] State variables can be packed to use fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L28-L55

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
