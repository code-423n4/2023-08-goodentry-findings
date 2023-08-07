# Gas Optimization

# Summary

| Number | Details                                                                                | issue |
| :----: | :------------------------------------------------------------------------------------- | :---: |
| [G-01] | Sort Solidity operations using short-circuit mode                                      |   2   |
| [G-02] | Avoid contract existence checks by using low level calls                               |   6   |
| [G-03] | Amounts should be checked for 0 before calling a transfer                              |   9   |
| [G-04] | Using delete statement can save gas                                                    |   1   |
| [G-05] | Using private rather than public for Constants, Saves Gas                              |   6   |
| [G-06] | Not using the named return variable when a function returns, wastes deployment gas     |   4   |
| [G-07] | Use calldata instead of memory for function arguments that do not get mutated          |   5   |
| [G-08] | Can Make The Variable Outside The Loop To Save Gas                                     |   5   |
| [G-09] | Internal functions not called by the contract should be removed to save deployment gas |   4   |
| [G-10] | Should use arguments instead of state variable                                         |   1   |
| [G-11] | Require() statements that check input arguments should be at the top of the function   |   6   |
| [G-12] | Empty blocks should be removed or emit something                                       |   2   |
| [G-13] | With assembly, .call (bool success) transfer can be done gas-optimized                 |   3   |
| [G-14] | Use assembly to write address storage values                                           |   5   |
| [G-15] | Caching global variables is more expensive than using the actual variable              |   1   |
| [G-16] | Duplicated require()/if() checks should be refactored to a modifier or function        |   5   |
| [G-17] | Use Assembly To Check For address(0)                                                   |   1   |
| [G-18] | abi.encode() is less efficient than abi.encodepacked()                                 |   2   |
| [G-19] | Require()/Revert() string longer than 32 bytes cost extra gas                          |   1   |
| [G-20] | SPLITTING  REQUIRE() STATEMENTS THAT USE  && SAVES GAS                                 |   7   |
| [G-21] | State variables can be packed to use fewer storage slots                               |   1   |

## [G-01] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.
2

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

## [G-02] Avoid contract existence checks by using low level call4

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

135    (uint256 amount0, uint256 amount1) = TokenisableRange(flashAsset).withdraw(flashAmount, 0, 0);

270      (uint token0Amount, uint token1Amount) = TokenisableRange(debtAsset).getTokenAmounts(debt);

305      debt = TokenisableRange(debtAsset).deposit(token0Amount, token1Amount);

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L135

```solidity
file: /contracts/GeVault.sol

87    lendingPool = ILendingPool(ILendingPoolAddressesProvider(lpap).getLendingPool());

88    oracle = IPriceOracle(ILendingPoolAddressesProvider(lpap).getPriceOracle());

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L87-L88

```solidity
file: /contracts/TokenisableRange.sol

370    (uint160 sqrtPriceX96,,,,,,)  = IUniswapV3Pool(pool).slot0();

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L370

## [G-03] Amounts should be checked for 0 before calling a transfer

```solidity
File:  contracts/helper/V3Proxy.sol

115  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);

164  ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);

182  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L115

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

## [G-04] Using delete statement can save gas

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

204      flashtype[i] = 0; // dont open debt for liquidations, need to repay

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L204

## [G-05] Using private rather than public for Constants, Saves Gas

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

## [G-06] Not using the named return variable when a function returns, wastes deployment gas

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

## [G-07] Use calldata instead of memory for function arguments that do not get mutated

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.

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
file: /contracts/TokenisableRange.sol

85     function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L85

```solidity
file: /contracts/RangeManager.sol

75    function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75

## [G-08] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

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

## [G-09] Internal functions not called by the contract should be removed to save deployment gas

When you define an internal function in a Solidity contract, Solidity generates code for that function and includes it in the contract bytecode. If the function is not called by the contract itself or any of its external functions, then

```solidity
File:  contracts/PositionManager/PositionManager.sol


63    function getPoolAddresses(uint poolId)
    internal view
    returns( ILendingPool lp, IPriceOracle oracle, IUniswapV2Router01 router, address token0, address token1)
  {



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

## [G-10] Should use arguments instead of state variable

state variables should not used in emit , This will save near 97 gas

```solidity
file: /contracts/GeVault.sol

362    emit Rebalance(tickIndex);

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362

## [G-11] Require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas\*) in a function that may ultimately revert in the unhappy case.

```solidity
file: /contracts/TokenisableRange.sol

209      require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L209

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

## [G-12] Empty blocks should be removed or emit something

f the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) .
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

```solidity
file: /contracts/PositionManager/PositionManager.sol

52       ) virtual external returns (bool result) {
53          }

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L52-L53

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

25      constructor (address roerouter) PositionManager(roerouter) {}

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L25

## [G-13] With assembly, .call (bool success) transfer can be done gas-optimized

Return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity
File:  contracts/helper/V3Proxy.sol

156   msg.sender.call{value: msg.value - amounts[0]}("");

174   payable(msg.sender).call{value: amountOut}("");

192   payable(msg.sender).call{value: amounts[1]}("");
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156

## [G-14] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

```solidity
File:  contracts/RoeRouter.sol

35   treasury = treasury_;
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L35

```solidity
File:  contracts/GeVault.sol

89   treasury = _treasury;

90   uniswapPool = IUniswapV3Pool(_uniswapPool);

91    WETH = IWETH(weth);

109   treasury = newTreasury;
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L89

## [G-15] Caching global variables is more expensive than using the actual variable

```solidity
file: /contracts/TokenisableRange.sol

88    creator = msg.sender;

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L88

## [G-16] Duplicated require()/if() checks should be refactored to a modifier or function

to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

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


120     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
141     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
170     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");


203    require(poolMatchesOracle(), "GEV: Oracle Error");
215    require(poolMatchesOracle(), "GEV: Oracle Error");
250    require(poolMatchesOracle(), "GEV: Oracle Error");
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120

```solidity
file: /contracts/GeVault.sol


121     if (ticks.length == 0) ticks.push(t);
142     if (ticks.length == 0) ticks.push(t);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L121

## [G-17] Use Assembly To Check For address(0)

```solidity
File:  contracts/PositionManager/OptionsPositionManager.sol

136   if (sourceSwap != address(0) ){
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L136

## [G-18] abi.encode() is less efficient than abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

168    bytes memory params = abi.encode(0, poolId, msg.sender, sourceSwap);

199    bytes memory params = abi.encode(1, poolId, user, collateralAsset); // mode = 1 -> liquidation

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L168

## [G-19] Require()/Revert() string longer than 32 bytes cost extra gas

when these strings get longer than 32 bytes, they cost more gas
Use shorter require()/revert() strings

```solidity
file: /contracts/helper/FixedOracle.sol

11        require(msg.sender == owner, "Only the owner can call this function.");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11

## [G-20] SPLITTING  REQUIRE() STATEMENTS THAT USE  && SAVES GAS

Instead of using operator && on a single require. Using a two require can save more gas.
i.e. for require(version == 1 && bytecodeHash[1] == bytes1(0) , “zf”) ; use:
require(version == 1); require(\_bytecodeHash[1] == bytes(0));

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L68-L74

```solidity
file: /contracts/helper/LPOracle.sol

29    require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L29

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

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L167

```solidity
file: /contracts/GeVault.sol

141    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L141

```solidity
file: /contracts/RangeManager.sol

108    require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L108

## [G-21] State variables can be packed to use fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

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