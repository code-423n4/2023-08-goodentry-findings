
## Low Risk Issues
|       |Issue  |Instances|
|:-----:|:-----:|:-------:|
|L-01|Pool cannot be restore once deprecated|1|
|L-02|Function works different than comments declare|1|
|L-03|Setters should have initial value check|6|

## Not Critical Issues
|       |Issue  |Instances|
|:-----:|:-----:|:-------:|
|NC-01|Lines are too long|60|
|NC-02|Empty Function Body. Considering commenting why.|1|
|NC-03|Constant redefined elsewhere|1|
|NC-04|Duplicated ```require()```/```revert()``` checks should be refactored to a modifier or function|16|
|NC-05|Control structures do not follow the Solidity Style Guide|21|
|NC-06|Events are missing sender information|3|
|NC-07|Long functions should be refactored into multiple functions|2|

# Low Risk Issues
## L-01
### Pool cannot be restored once deprecated
#### Description
Owner can set pool as deprecated. He/she could do that when pool has not enough liquidity, using

```
File: RoeRouter.sol

45  /// @notice Deprecate a pool
46  /// @param poolId pool ID
47  /// @dev isDeprecated is a statement about the pool record, and does not imply anything about the pool itself
48  function deprecatePool(uint poolId) public onlyOwner {
49      pools[poolId].isDeprecated = true;
50      emit DeprecatePool(poolId);
51  }

```
[[45-51](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L45-L51)]

However, there's no way to restore a pool, i.e. no function set ```pools[poolId].isDeprecated = false```

## L-02
### Function works different than comments declare
#### Description
In OptionsPositionManager.sol, executeOperation()'s comments declare ```@return result True if the execution of the operation succeeds, false otherwise```

```
File: OptionsPositionManager.sol

29  /**
30  * @notice Aave-compatible flashloan receiver dispatch: open a leverage position or liquidate a position
31  * @param assets The address of the flash-borrowed asset
32  * @param amounts The amount of the flash-borrowed asset
33  * @param premiums The fee of the flash-borrowed asset
34  * @param initiator The address of the flashloan initiator
35  * @param params The byte-encoded params passed when initiating the flashloan
36  * @return result True if the execution of the operation succeeds, false otherwise
37  */
38  function executeOperation(
39      address[] calldata assets,
40      uint256[] calldata amounts,
41      uint256[] calldata premiums,
42      address initiator,
43      bytes calldata params
44  ) override external returns (bool result) {
45      uint8 mode = abi.decode(params, (uint8) );
46      // Buy options
47      if ( mode == 0 ){
48          (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));
49          executeBuyOptions(poolId, assets, amounts, user, sourceSwap);
50      }
51      // Liquidate
52      else {
53          (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));
54          executeLiquidation(poolId, assets, amounts, user, collateral);
55      }
56      result = true;
57  }

```
[[29-57](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L29-L57)]

However, this function returns always ```true```

## L-03
### Setters should have initial value check
#### Description
Setters should have initial value check to prevent assigning wrong value to the variable. Assignment of wrong value can lead to unexpected behavior of the contract.


<details>
<summary>There are 6 instances of this issue:</summary>

```
File: FixedOracle.sol

  @audit: unchecked values _hardcodedPrice
  24     function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {
  25         hardcodedPrice = _hardcodedPrice;
  26         emit SetHardcodedPrice(_hardcodedPrice);
  27     }

```
[[24-27](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24-L27)]

```
File: GeVault.sol

  @audit: unchecked values _isEnabled
  101   function setEnabled(bool _isEnabled) public onlyOwner { 
  102     isEnabled = _isEnabled; 
  103     emit SetEnabled(_isEnabled);
  104   }

  @audit: unchecked values newTreasury
  108   function setTreasury(address newTreasury) public onlyOwner { 
  109     treasury = newTreasury; 
  110     emit SetTreasury(newTreasury);
  111   }

  @audit: unchecked values newBaseFeeX4
  183   function setBaseFee(uint newBaseFeeX4) public onlyOwner {
  184   require(newBaseFeeX4 < 1e4, "GEV: Invalid Base Fee");
  185     baseFeeX4 = newBaseFeeX4;
  186     emit SetFee(newBaseFeeX4);
  187   }

  @audit: unchecked values newTvlCap
  191   function setTvlCap(uint newTvlCap) public onlyOwner {
  192     tvlCap = newTvlCap;
  193     emit SetTvlCap(newTvlCap);
  194   }

```
[[101-104](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101-L104), [108-111](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L108-L111), [183-187](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L183-L187), [191-194](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L191-L194)]


```
File: RoeRouter.sol

  @audit: unchecked values newTreasury
  83   function setTreasury(address newTreasury) public onlyOwner {
  84     require(newTreasury != address(0x0), "Invalid address");
  85     treasury = newTreasury;
  86     emit UpdateTreasury(newTreasury);
  87   }

```
[[83-87](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L83-L87)]

            
</details>



# Not Critical Issues
## NC-01
### Lines are too long
#### Description
Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. The solidity style guide recommends a maximumum line length of 120 characters, so the lines below should be split when they reach that length.


<details>
<summary>There are 60 instances of this issue:</summary>

```
File: GeVault.sol

  
  213   /// @dev For simplicity+efficieny, withdrawal is like a rebalancing, but a subset of the tokens are sent back to the user before redeploying

  
  386     if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

```
[[213](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L213), [386](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L386)]

```
File: IAaveIncentivesController.sol

  
  105    * @dev Claims reward for an user on behalf, on all the assets of the lending pool, accumulating the pending rewards. The caller must

```
[[105](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L105)]

```
File: IAaveLendingPoolV2.sol

  
  130    * @param collateralAsset The address of the underlying asset used as collateral, to receive as result of the liquidation

  
  258    *     2. the current deposit APY is below REBALANCE_UP_THRESHOLD * maxVariableBorrowRate, which means that too much has been

  
  276    * @param collateralAsset The address of the underlying asset used as collateral, to receive as result of the liquidation

  
  294    * IMPORTANT There are security concerns for developers of flashloan receiver contracts that must be kept into consideration.

  
  296    * @param receiverAddress The address of the contract receiving the funds, implementing the IFlashLoanReceiver interface

```
[[130](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveLendingPoolV2.sol#L130), [258](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveLendingPoolV2.sol#L258), [276](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveLendingPoolV2.sol#L276), [294](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveLendingPoolV2.sol#L294), [296](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveLendingPoolV2.sol#L296)]

```
File: ILendingPoolConfigurator.sol

  
  91    * @param liquidationThreshold The threshold at which loans using this asset as collateral will be considered undercollateralized

```
[[91](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ILendingPoolConfigurator.sol#L91)]

```
File: IPoolInitializer.sol

  
  10     /// @dev This method can be bundled with others via IMulticall for the first action (e.g. mint) performed against a pool

  
  15     /// @return pool Returns the pool address based on the pair of tokens and fee, will return the newly created pool address if necessary

```
[[10](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IPoolInitializer.sol#L10), [15](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IPoolInitializer.sol#L15)]

```
File: LPOracle.sol

  
  77       norm_a, norm_b represents a, b adjusted along the k curve such that it represents the amounts the uniswap pool will contain at the Chainlink oracle price 

```
[[77](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L77)]

```
File: OptionsPositionManager.sol

  
  48       (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));

  
  144       // if swap underlying, then sourceSwap amount is 0 and the other amount is amount withdrawn + amount received from swap

  
  158   /// @dev Because tickers are one coin or the other depending on the price, one can only buy OTM options. You can get ITM put by buying OTM call and swapping, or get ITM call by buying OTM put and swapping. If you wanna swap, set sourceSwap to the asset you *dont* want, otherwise must be address(0x0)

  
  246   /// @param user Owner of the debt to close. If user is address(this), we dont repay but just recreate tokens, flashloan will take care of getting them back

  
  314       // edge case where after swapping exactly the tokens and repaying debt, dust causes remaining asset balance to be slightly higher than before repaying

  
  331   /// @notice Check that amounts to deposit in TR are matching expected balance based on oracle, to avoid sandwich attacks

  
  342     uint tokensValue = token0Amount * oracle.getAssetPrice(address(token0)) / 10**decimals0 + token1Amount * oracle.getAssetPrice(address(token1)) / 10**decimals1;

  
  440     (ILendingPool LP, IPriceOracle oracle,IUniswapV2Router01 router, address token0, address token1) = getPoolAddresses(poolId);

  
  470   function swapExactTokensForTokens(IUniswapV2Router01 ammRouter, IPriceOracle oracle, uint amount, address[] memory path) 

  
  491   function swapTokensForExactTokens(IUniswapV2Router01 ammRouter, uint recvAmount, uint maxAmount, address[] memory path) internal {

```
[[48](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L48), [144](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L144), [158](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L158), [246](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L246), [314](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L314), [331](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L331), [342](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L342), [440](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L440), [470](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L470), [491](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L491)]

```
File: OracleConvert.sol

  
  78   function latestRoundData() external view returns (uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound) {

```
[[78](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L78)]

```
File: PositionManager.sol

  
  81     if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

  
  138   function validateValuesAgainstOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB, uint amountB) internal view {

```
[[81](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L81), [138](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L138)]

```
File: RangeManager.sol

  
  36   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);  

  
  75   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

  
  83     IAaveOracle oracle = IAaveOracle(ILendingPoolAddressesProvider( LENDING_POOL.getAddressesProvider() ).getPriceOracle());

  
  85     tokenisedRanges[ tokenisedRanges.length - 1 ].initProxy(oracle, ASSET_0, ASSET_1, startX10, endX10, startName, endName, false);

  
  86     tokenisedTicker[ tokenisedTicker.length - 1 ].initProxy(oracle, ASSET_0, ASSET_1, startX10, endX10, startName, endName, true); 

```
[[36](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36), [75](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75), [83](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L83), [85](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L85), [86](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L86)]

```
File: RoeRouter.sol

  
  58   /// @param ammRouter address of the AMMv2 such that the LP pair ammRouter.factory.getPair(token0, token1) is supported by the lending pool

```
[[58](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L58)]

```
File: TokenisableRange.sol

  
  58   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88); 

  
  85   function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {

  
  99     int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );

  
  100     int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );

  
  209       require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

  
  233     // Calculate proportion of deposit that goes to pending fee pool, useful to deposit exact amount of liquidity and fully repay a position

  
  236       // DeMorgan: !( (n0 == 0 && fee0 > 0) || (n1 == 0 && fee1 > 0) ) = !(n0 == 0 && fee0 > 0) && !(n0 == 0 && fee1 > 0)

  
  240       (uint256 token0Amount, uint256 token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick), liquidity);

  
  273       // Assume linearity for liquidity in same tick range; calculate feeLiquidity equivalent and consider it part of base liquidity 

  
  274       feeLiquidity = newLiquidity * ( (fee0 * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (fee1 * TOKEN1_PRICE / 10 ** TOKEN1.decimals) )   

  
  275                                     / ( (added0   * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (added1   * TOKEN1_PRICE / 10 ** TOKEN1.decimals) ); 

  
  292   function withdraw(uint256 lp, uint256 amount0Min, uint256 amount1Min) external nonReentrant returns (uint256 removed0, uint256 removed1) {

  
  333   function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {

  
  338     (amt0, amt1) = LiquidityAmounts.getAmountsForLiquidity( uint160( sqrt( (2 ** 192 * ((TOKEN0_PRICE * 10 ** TOKEN1.decimals) / TOKEN1_PRICE)) / ( 10 ** TOKEN0.decimals ) ) ), TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  liquidity);

  
  343   function returnExpectedBalance(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 amt0, uint256 amt1) {

  
  362     return getValuePerLPAtPrice(ORACLE.getAssetPrice(address(TOKEN0.token)), ORACLE.getAssetPrice(address(TOKEN1.token)));

  
  371     (token0Amount, token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  uint128 ( uint(liquidity) * amount / totalSupply() ) );

```
[[58](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L58), [85](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L85), [99](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L99), [100](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L100), [209](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L209), [233](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L233), [236](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L236), [240](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L240), [273](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L273), [274](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L274), [275](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L275), [292](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L292), [333](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333), [338](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L338), [343](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L343), [362](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L362), [371](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L371)]

```
File: V3Proxy.sol

  
  112     function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

  
  119         amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountIn, amountOutMin, 0));

  
  124     function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

  
  130         amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountOut, amountInMax, 0));         

  
  137     function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

  
  142         amounts[1] = ROUTER.exactInputSingle{value: msg.value}(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, msg.value, amountOutMin, 0));

  
  147     function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

  
  151         amounts[0] = ROUTER.exactOutputSingle{value: msg.value}(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountOut, msg.value, 0));         

  
  160     function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

  
  167         amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountOut, amountInMax, 0));         

  
  178     function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

  
  186         amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountIn, amountOutMin, 0));

```
[[112](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L112), [119](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L119), [124](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L124), [130](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L130), [137](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L137), [142](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L142), [147](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L147), [151](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L151), [160](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L160), [167](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L167), [178](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L178), [186](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L186)]

            
</details>

## NC-02
### Empty Function Body. Considering commenting why.
#### Description
There are functions without documentation


There is 1 instance of this issue:                

```
File: OptionsPositionManager.sol

  
  25   constructor (address roerouter) PositionManager(roerouter) {}

```
[[25](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L25)]


## NC-03
### Constant redefined elsewhere
#### Description
Consider defining in only one contract so that values cannot become out of sync when only one location is updated. A [cheap way](https://medium.com/coinmonks/gas-cost-of-solidity-library-functions-dbe0cedd4678) to store constants in a single location is to create an internal constant in a library. If the variable is a local cache of another contract's value, consider making the cache variable internal or private, which will require external users to query the contract with the source of truth, so that callers don't get out of sync.


There is 1 instance of this issue:                

```
File: RangeManager.sol

  @audit: Founded in TokenisableRange, RangeManager
  36   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);  

```
[[36](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36)]
## NC-04
### Duplicated ```require()```/```revert()``` checks should be refactored to a modifier or function
#### Description
The compiler will inline the function, which will avoid ```JUMP``` instructions usually associated with functions


<details>
<summary>There are 16 instances of this issue:</summary>

```
File: GeVault.sol

  
  141     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

  
  170     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

  
  215     require(poolMatchesOracle(), "GEV: Oracle Error");

  
  250     require(poolMatchesOracle(), "GEV: Oracle Error");

```
[[141](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L141), [170](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L170), [215](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L215), [250](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L250)]

```
File: OptionsPositionManager.sol

  
  91     require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");

  
  420     require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

```
[[91](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L91), [420](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L420)]

```
File: V3Proxy.sol

  
  91        require(acceptPayable, "CannotReceiveETH");

  
  106         require(path.length == 2, "Direct swap only");

  
  113         require(path.length == 2, "Direct swap only");

  
  125         require(path.length == 2, "Direct swap only");

  
  138         require(path.length == 2, "Direct swap only");

  
  148         require(path.length == 2, "Direct swap only");

  
  149         require(path[0] == ROUTER.WETH9(), "Invalid path");

  
  161         require(path.length == 2, "Direct swap only");

  
  179         require(path.length == 2, "Direct swap only");

  
  180         require(path[1] == ROUTER.WETH9(), "Invalid path");

```
[[91](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L91), [106](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L106), [113](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L113), [125](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L125), [138](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L138), [148](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L148), [149](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L149), [161](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L161), [179](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L179), [180](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L180)]

            
</details>

## NC-05
### Control structures do not follow the Solidity Style Guide
#### Description
See the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) section of the Solidity Style Guide


<details>
<summary>There are 21 instances of this issue:</summary>

```
File: GeVault.sol

  
  153       if (ticks.length > 2){
  154         for (uint k = 0; k < ticks.length - 2; k++) 
  155           ticks[ticks.length - 2 - k] = ticks[ticks.length - 3 - k];
  156         }

  
  230     if (token == address(WETH)){
  231       WETH.withdraw(bal);
  232       payable(msg.sender).transfer(bal);
  233     }
  234     else {
  235       ERC20(token).safeTransfer(msg.sender, bal);
  236     }

  
  255     if (msg.value > 0){
  256       require(token == address(WETH), "GEV: Invalid Weth");
  257       // wraps ETH by sending to the wrapper that sends back WETH
  258       WETH.deposit{value: msg.value}();
  259       amount = msg.value;
  260     }
  261     else { 
  262       ERC20(token).safeTransferFrom(msg.sender, address(this), amount);
  263     }

  
  329     if (aBal > 0){
  330       lendingPool.withdraw(address(tr), aBal, address(this));
  331       tr.withdraw(aBal, 0, 0);
  332     }

  
  346     if (amount1ft > 0){
  347       tick0Index = newTickIndex + 2;
  348       tick1Index = newTickIndex;
  349     }

  
  352     if (availToken0 > 0){
  353       depositAndStash(ticks[tick0Index], availToken0 / 2, 0);
  354       depositAndStash(ticks[tick0Index+1], availToken0 / 2, 0);
  355     }

  
  356     if (availToken1 > 0){
  357       depositAndStash(ticks[tick1Index], 0, availToken1 / 2);
  358       depositAndStash(ticks[tick1Index+1], 0, availToken1 / 2);
  359     }

  
  410     if (bal > 0){
  411       checkSetApprove(address(t), address(lendingPool), bal);
  412       lendingPool.deposit(address(t), bal, address(this), 0);
  413     }

  
  429     if (ticks.length >= 5){
  430       // looking for index at which the underlying asset differs from the next tick
  431       for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){
  432         (uint amt0, uint amt1) = ticks[activeTickIndex+1].getTokenAmountsExcludingFees(1e18);
  433         (uint amt0n, uint amt1n) = ticks[activeTickIndex+2].getTokenAmountsExcludingFees(1e18);
  434         if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )
  435           break;
  436       }
  437     }

  
  463     if(msg.sender != address(WETH)) deposit(address(WETH), msg.value);

```
[[153-156](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L153-L156), [230-236](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L230-L236), [255-263](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L255-L263), [329-332](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L329-L332), [346-349](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L346-L349), [352-355](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L352-L355), [356-359](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L356-L359), [410-413](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L410-L413), [429-437](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L429-L437), [463](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L463)]

```
File: INonfungiblePositionManager.sol

  
  17 interface INonfungiblePositionManager is
  18     IPoolInitializer,
  19     IPeripheryPayments,
  20     IPeripheryImmutableState,
  21     IERC721Metadata,
  22     IERC721Enumerable,
  23     IERC721Permit
  24 {
  25     /// @notice Emitted when liquidity is increased for a position NFT
  26     /// @dev Also emitted when a token is minted
  27     /// @param tokenId The ID of the token for which liquidity was increased
  28     /// @param liquidity The amount by which liquidity for the NFT position was increased
  29     /// @param amount0 The amount of token0 that was paid for the increase in liquidity
  30     /// @param amount1 The amount of token1 that was paid for the increase in liquidity

```
[[17-30](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L17-L30)]

```
File: OptionsPositionManager.sol

  
  47     if ( mode == 0 ){
  48       (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));
  49       executeBuyOptions(poolId, assets, amounts, user, sourceSwap);
  50     }
  51     // Liquidate
  52     else {
  53       (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));
  54       executeLiquidation(poolId, assets, amounts, user, collateral);
  55     }

  
  136     if (sourceSwap != address(0) ){
  137       require(sourceSwap == token0 || sourceSwap == token1, "OPM: Invalid Swap Token");
  138       address[] memory path = new address[](2);
  139       path[0] = sourceSwap ;
  140       path[1] = sourceSwap == token0 ? token1 : token0;
  141       uint amount = sourceSwap == token0 ? amount0 : amount1;
  142 
  143       uint received = swapExactTokensForTokens(router, oracle, amount, path);
  144       // if swap underlying, then sourceSwap amount is 0 and the other amount is amount withdrawn + amount received from swap
  145       amount0 = sourceSwap == token0 ? 0 : amount0 + received;
  146       amount1 = sourceSwap == token1 ? 0 : amount1 + received;
  147     }

  
  275       if (user != address(this) ){
  276         amtA = IERC20(LP.getReserveData(token0).aTokenAddress ).balanceOf(user);
  277         amtB = IERC20(LP.getReserveData(token1).aTokenAddress ).balanceOf(user);
  278         PMWithdraw(LP, user, token0, amtA );
  279         PMWithdraw(LP, user, token1, amtB );
  280         // If another user softLiquidates a share of the liquidation goes to the treasury
  281         if (user != msg.sender ) {
  282           uint feeAmount = calculateAndSendFee(poolId, token0Amount, token1Amount, collateralAsset);
  283           if (collateralAsset == token0) amtA -= feeAmount;
  284           else amtB -= feeAmount;
  285         }
  286       }
  287       else {
  288         // Assets are already present from liquidation
  289         amtA = ERC20(token0).balanceOf(user);
  290         amtB = ERC20(token1).balanceOf(user);
  291       }

  
  295       if ( amtA < token0Amount ){
  296         path[0] = token1;
  297         path[1] = token0;
  298         swapTokensForExactTokens(ammRouter, token0Amount - amtA, amtB, path); 
  299       }
  300       else if ( amtB < token1Amount ){
  301         path[0] = token0;
  302         path[1] = token1;
  303         swapTokensForExactTokens(ammRouter, token1Amount - amtB, amtA, path); 
  304       }

  
  300       else if ( amtB < token1Amount ){
  301         path[0] = token0;
  302         path[1] = token1;
  303         swapTokensForExactTokens(ammRouter, token1Amount - amtB, amtA, path); 
  304       }

  
  473     if (amount > 0 && AmountsRouter(address(ammRouter)).getAmountsOut(amount, path)[1] > 0){
  474       checkSetAllowance(path[0], address(ammRouter), amount);
  475       uint[] memory amounts = ammRouter.swapExactTokensForTokens(
  476         amount, 
  477         getTargetAmountFromOracle(oracle, path[0], amount, path[1]) * 99 / 100, // allow 1% slippage 
  478         path, 
  479         address(this), 
  480         block.timestamp
  481       );
  482       received = amounts[1];
  483     }

```
[[47-55](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L47-L55), [136-147](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L136-L147), [275-291](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L275-L291), [295-304](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L295-L304), [300-304](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L300-L304), [473-483](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L473-L483)]

```
File: PositionManager.sol

  
  96       if ( debt > 0 ){
  97         if (amt <= debt ) {
  98           LP.repay( asset, amt, 2, user);
  99           return;
  100         }
  101         else {
  102           LP.repay( asset, debt, 2, user);
  103           amt = amt - debt;
  104         }
  105       }

  
  125     if ( amount > 0 ){
  126       LP.PMTransfer(LP.getReserveData(asset).aTokenAddress, user, amount);
  127       LP.withdraw(asset, amount, address(this));
  128     }

```
[[96-105](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L96-L105), [125-128](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L125-L128)]

```
File: TokenisableRange.sol

  
  237     if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) ){
  238       address pool = V3_FACTORY.getPool(address(TOKEN0.token), address(TOKEN1.token), feeTier * 100);
  239       (uint160 sqrtPriceX96,,,,,,)  = IUniswapV3Pool(pool).slot0();
  240       (uint256 token0Amount, uint256 token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick), liquidity);
  241       if (token0Amount + fee0 > 0) newFee0 = n0 * fee0 / (token0Amount + fee0);
  242       if (token1Amount + fee1 > 0) newFee1 = n1 * fee1 / (token1Amount + fee1);
  243       fee0 += newFee0;
  244       fee1 += newFee1; 
  245       n0   -= newFee0;
  246       n1   -= newFee1;
  247     }

  
  268     if ( newFee0 == 0 && newFee1 == 0 ){
  269       uint256 TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token));
  270       uint256 TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token));
  271       require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");
  272       // Calculate the equivalent liquidity amount of the non-yet compounded fees
  273       // Assume linearity for liquidity in same tick range; calculate feeLiquidity equivalent and consider it part of base liquidity 
  274       feeLiquidity = newLiquidity * ( (fee0 * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (fee1 * TOKEN1_PRICE / 10 ** TOKEN1.decimals) )   
  275                                     / ( (added0   * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (added1   * TOKEN1_PRICE / 10 ** TOKEN1.decimals) ); 
  276     }

```
[[237-247](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L237-L247), [268-276](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L268-L276)]

            
</details>

## NC-06
### Events are missing sender information
#### Description
When an action is triggered based on a user's action, not being able to filter based on who triggered the action makes event processing a lot more cumbersome. Including the msg.sender the events of these types of action will make events much more useful to end users, especially when ```msg.sender``` is not ```tx.origin```.


There are 3 instances of this issue:                

```
File: OptionsPositionManager.sol

  
  106       emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1]);

  
  323       emit ClosePosition(user, debtAsset, debt, amt0, amt1);

```
[[106](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L106), [323](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L323)]

```
File: TokenisableRange.sol

  
  119     emit InitTR(address(asset0), address(asset1), startX10, endX10);

```
[[119](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L119)]


## NC-07
### Long functions should be refactored into multiple functions
#### Description
Consider splitting long functions into multiple, smaller functions to improve the code readability.


There are 2 instances of this issue:                

```
File: OptionsPositionManager.sol

  
  250   function closeDebt(
  251     uint poolId, 
  252     address user,
  253     address debtAsset, 
  254     uint repayAmount,
  255     address collateralAsset
  256   ) 
  257     internal returns (uint debt)
  258   {
  259     (ILendingPool LP,,IUniswapV2Router01 ammRouter, address token0, address token1) = getPoolAddresses(poolId);
  260     sanityCheckUnderlying(debtAsset, token0, token1);
  261     require(collateralAsset == token0 || collateralAsset == token1, "OPM: Invalid Collateral Asset");
  262     uint amtA;
  263     uint amtB;
  264     // Add dust to be sure debt reformed >= debt outstanding
  265     debt = repayAmount + addDust(debtAsset, token0, token1);
  266     
  267     // Claim fees first so that deposit will match exactly
  268     TokenisableRange(debtAsset).claimFee();
  269     { //localize vars
  270       (uint token0Amount, uint token1Amount) = TokenisableRange(debtAsset).getTokenAmounts(debt);
  271       checkExpectedBalances(debtAsset, debt, token0Amount, token1Amount);
  272       checkSetAllowance(token0, debtAsset, token0Amount);
  273       checkSetAllowance(token1, debtAsset, token1Amount);
  274       // If called by this contract himself this is a liquidation, skip that step
  275       if (user != address(this) ){
  276         amtA = IERC20(LP.getReserveData(token0).aTokenAddress ).balanceOf(user);
  277         amtB = IERC20(LP.getReserveData(token1).aTokenAddress ).balanceOf(user);
  278         PMWithdraw(LP, user, token0, amtA );
  279         PMWithdraw(LP, user, token1, amtB );
  280         // If another user softLiquidates a share of the liquidation goes to the treasury
  281         if (user != msg.sender ) {
  282           uint feeAmount = calculateAndSendFee(poolId, token0Amount, token1Amount, collateralAsset);
  283           if (collateralAsset == token0) amtA -= feeAmount;
  284           else amtB -= feeAmount;
  285         }
  286       }
  287       else {
  288         // Assets are already present from liquidation
  289         amtA = ERC20(token0).balanceOf(user);
  290         amtB = ERC20(token1).balanceOf(user);
  291       }
  292 
  293       // swap if one token is missing - consider that there is enough 
  294       address[] memory path = new address[](2);
  295       if ( amtA < token0Amount ){
  296         path[0] = token1;
  297         path[1] = token0;
  298         swapTokensForExactTokens(ammRouter, token0Amount - amtA, amtB, path); 
  299       }
  300       else if ( amtB < token1Amount ){
  301         path[0] = token0;
  302         path[1] = token1;
  303         swapTokensForExactTokens(ammRouter, token1Amount - amtB, amtA, path); 
  304       }
  305       debt = TokenisableRange(debtAsset).deposit(token0Amount, token1Amount);
  306     }
  307     checkSetAllowance(debtAsset, address(LP), debt);
  308     
  309     // If user closes, repay debt, else tokens will be taken back by the flashloan
  310     if (user != address(this) ) LP.repay( debtAsset, debt, 2, user);
  311     {
  312       uint amt0 = ERC20(token0).balanceOf(address(this));
  313       uint amt1 = ERC20(token1).balanceOf(address(this));
  314       // edge case where after swapping exactly the tokens and repaying debt, dust causes remaining asset balance to be slightly higher than before repaying
  315       if (amtA > amt0) 
  316         amt0 = amtA - amt0;
  317       else 
  318         amt0 = 0;
  319       if (amtB > amt1) 
  320         amt1 = amtB - amt1;
  321       else 
  322         amt1 = 0;
  323       emit ClosePosition(user, debtAsset, debt, amt0, amt1);
  324     }
  325     
  326     // Swap other token back to collateral: this allows to control exposure
  327     if (user == msg.sender) swapTokens(poolId, collateralAsset == token0 ? token1 : token0, 0);
  328   }

```
[[250-328](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L250-L328)]

```
File: TokenisableRange.sol

  
  222   function deposit(uint256 n0, uint256 n1) external nonReentrant returns (uint256 lpAmt) {
  223     // Once all assets were withdrawn after initialisation, this is considered closed
  224     // Prevents TR oracle values from being too manipulatable by emptying the range and redepositing 
  225     require(totalSupply() > 0, "TR Closed"); 
  226     
  227     claimFee();
  228     TOKEN0.token.transferFrom(msg.sender, address(this), n0);
  229     TOKEN1.token.transferFrom(msg.sender, address(this), n1);
  230     
  231     uint newFee0; 
  232     uint newFee1;
  233     // Calculate proportion of deposit that goes to pending fee pool, useful to deposit exact amount of liquidity and fully repay a position
  234     // Cannot repay only one side, if fees are both 0, or if one side is missing, skip adding fees here
  235       // if ( fee0+fee1 == 0 || (n0 == 0 && fee0 > 0) || (n1 == 0 && fee1 > 0) ) skip  
  236       // DeMorgan: !( (n0 == 0 && fee0 > 0) || (n1 == 0 && fee1 > 0) ) = !(n0 == 0 && fee0 > 0) && !(n0 == 0 && fee1 > 0)
  237     if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) ){
  238       address pool = V3_FACTORY.getPool(address(TOKEN0.token), address(TOKEN1.token), feeTier * 100);
  239       (uint160 sqrtPriceX96,,,,,,)  = IUniswapV3Pool(pool).slot0();
  240       (uint256 token0Amount, uint256 token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick), liquidity);
  241       if (token0Amount + fee0 > 0) newFee0 = n0 * fee0 / (token0Amount + fee0);
  242       if (token1Amount + fee1 > 0) newFee1 = n1 * fee1 / (token1Amount + fee1);
  243       fee0 += newFee0;
  244       fee1 += newFee1; 
  245       n0   -= newFee0;
  246       n1   -= newFee1;
  247     }
  248 
  249     TOKEN0.token.safeIncreaseAllowance(address(POS_MGR), n0);
  250     TOKEN1.token.safeIncreaseAllowance(address(POS_MGR), n1);
  251 
  252     // New liquidity is indeed the amount of liquidity added, not the total, despite being unclear in Uniswap doc
  253     (uint128 newLiquidity, uint256 added0, uint256 added1) = POS_MGR.increaseLiquidity(
  254       INonfungiblePositionManager.IncreaseLiquidityParams({
  255         tokenId: tokenId,
  256         amount0Desired: n0,
  257         amount1Desired: n1,
  258         amount0Min: n0 * 95 / 100,
  259         amount1Min: n1 * 95 / 100,
  260         deadline: block.timestamp
  261       })
  262     );
  263     
  264     uint256 feeLiquidity;
  265 
  266     // Stack too deep, so localising some variables for feeLiquidity calculations 
  267     // If we already clawed back fees earlier, do nothing, else we need to adjust returned liquidity
  268     if ( newFee0 == 0 && newFee1 == 0 ){
  269       uint256 TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token));
  270       uint256 TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token));
  271       require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");
  272       // Calculate the equivalent liquidity amount of the non-yet compounded fees
  273       // Assume linearity for liquidity in same tick range; calculate feeLiquidity equivalent and consider it part of base liquidity 
  274       feeLiquidity = newLiquidity * ( (fee0 * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (fee1 * TOKEN1_PRICE / 10 ** TOKEN1.decimals) )   
  275                                     / ( (added0   * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (added1   * TOKEN1_PRICE / 10 ** TOKEN1.decimals) ); 
  276     }
  277                                      
  278     lpAmt = totalSupply() * newLiquidity / (liquidity + feeLiquidity); 
  279     liquidity = liquidity + newLiquidity;
  280     
  281     _mint(msg.sender, lpAmt);
  282     TOKEN0.token.safeTransfer( msg.sender, n0 - added0);
  283     TOKEN1.token.safeTransfer( msg.sender, n1 - added1);
  284     emit Deposit(msg.sender, lpAmt);
  285   }

```
[[222-285](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L222-L285)]

