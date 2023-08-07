
## Gas Optimizations

| Number | Issue | Instances | Total gas saved |
|--------|-------|-----------|-----------------|
|[G-01]| State variables which are not modified within functions should be set as constant or immutable for values set at deployment   |  8  | 80000  |
|[G-02]| Use assembly in place of abi.decode to extract calldata values more efficiently   | 3 |   |
|[G-03]| Cache external calls outside of loop to avoid re-calling function on each iteration   | 3 |   |
|[G-04]| Use assembly to perform efficient back-to-back calls   | 10 |   |
|[G-05]| Use calldata instead of memory for function arguments that do not get mutated   | 2 | 720  |
|[G-06]| Use custom errors instead of require/assert   | 75 |  3750  |
|[G-07]| Functions guaranteed to revert when called by normal users can be marked payable   | 14 | 180  |
|[G-08]| Use assembly to emit events   | 35 | 1292  |
|[G-09]| Use assembly to write address storage values   | 2 | 148  |
|[G-10]| Use hardcoded address instead address(this)   | 47 |   |
|[G-11]| Use uint256(1)/uint256(2) instead for true and false boolean states   | 8 | 136800  |
|[G-12]| Expensive operation inside a for loop   | 1 |   |
|[G-13]| Use assembly to validate msg.sender   | 5 |   |
|[G-14]| Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function   | 3 | 84 |
|[G-15]| A modifier used only once and not being inherited should be inlined to save gas   | 1 |   |
|[G-16]| Use assembly for loops   | 1 |   |
|[G-17]| require()/revert() strings longer than 32 bytes cost extra gas  | 4 | 56  |
|[G-18]| Using private rather than public for constants, saves gas   | 6 |   |
|[G-19]| Amounts should be checked for 0 before calling a transfer   | 6 |   |
|[G-20]| Use constants instead of type(uintx).max   | 7 |   |
|[G-21]| Should use arguments instead of state variable    | 1 | 97  |
|[G-22]| Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)   | 1 |   |
|[G-23]| Empty blocks should be removed or emit something   | 2 |  8012  |
|[G-24]| Can make the variable outside the loop to save gas | 5 |   |
|[G-25]| abi.encode() is less efficient than  abi.encodepacked()   | 2 | 200  |
|[G-26]| Avoid contract existence checks by using low level calls	   | 6 | 600  |
|[G-27]| Using delete statement can save gas   | 1 |   |
|[G-28]| Not using the named return variable when a function returns, wastes deployment gas   | 5 |   |
|[G-29]| Sort Solidity operations using short-circuit mode   | 2 |  4200  |


## [G-01] State variables which are not modified within functions should be set as constant or immutable for values set at deployment

Cache such variables and perform operations on them, if operations include modifications to the state variable(s) then remember to equate the state variable to it's cached counterpart at the end


```solidity
file:  contracts/helper/FixedOracle.sol

7    address private owner;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L7

```solidity
file:  contracts/GeVault.sol

41     RangeManager rangeManager; 

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L41

```solidity
file: contracts/TokenisableRange.sol

54   address public TREASURY_DEPRECATED = 0x22Cc3f665ba4C898226353B672c5123c58751692;

55   uint public treasuryFee_deprecated = 20;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L54

## [G-02] Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```solidity
file: contracts/PositionManager/OptionsPositionManager.sol

45    uint8 mode = abi.decode(params, (uint8) );

48    (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));

53    (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L45

## [G-03] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

### Cache t.getTokenAmounts(bal) outside of loop to save 1 STATICCALL per loop iteration

```solidity
file:  main/contracts/GeVault.sol

299       for (uint k = 0; k < ticks.length; k++){
      TokenisableRange t = ticks[k];
      address aTick = lendingPool.getReserveData(address(t)).aTokenAddress;
      uint bal = ERC20(aTick).balanceOf(address(this));
      (uint amt0, uint amt1) = t.getTokenAmounts(bal);
      amount0 += amt0;
      amount1 += amt1;
    }

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L299-L306


### Cache  lendingPool.liquidationCall() outside of loop to save 1 STATICCALL per loop iteration

```solidity
file:  contracts/PositionManager/OptionsPositionManager.sol

93       for ( uint8 k =0; k<assets.length; k++){
      address debtAsset = assets[k];
      
      // simple liquidation: debt is transferred from user to liquidator and collateral deposited to roe
      uint amount = amounts[k];
      
      // liquidate and send assets here
      checkSetAllowance(debtAsset, address(lendingPool), amount);
      lendingPool.liquidationCall(collateral, debtAsset, user, amount, false);
      // repay tokens
      uint debt = closeDebt(poolId, address(this), debtAsset, amount, collateral);
      uint amt0 = ERC20(token0).balanceOf(address(this));
      uint amt1 = ERC20(token1).balanceOf(address(this));
      emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1]);
      amts[0] = amt0;
      amts[1] = amt1;
      
    }

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L93-L110


### Cache ticks[activeTickIndex+1].getTokenAmountsExcludingFees(1e18); outside of loop to save 2 STATICCALL per loop iteration

```solidity
file:   main/contracts/GeVault.sol

431        for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){
        (uint amt0, uint amt1) = ticks[activeTickIndex+1].getTokenAmountsExcludingFees(1e18);
        (uint amt0n, uint amt1n) = ticks[activeTickIndex+2].getTokenAmountsExcludingFees(1e18);
        if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )
          break;
      }

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L431-L436


## [G-04] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case, we are also able to efficiently store the function signatures together in memory as one word, saving multiple MLOADs in the process.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.


### use for this t.TOKEN0() back to bak  external call assembly 

```solidity
file:  contracts/GeVault.sol

116    function pushTick(address tr) public onlyOwner {
    TokenisableRange t = TokenisableRange(tr);
    (ERC20 t0,) = t.TOKEN0();
    (ERC20 t1,) = t.TOKEN1();
    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
    if (ticks.length == 0) ticks.push(t);
    else {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L116-L122

```solidity
file:  main/contracts/GeVault.sol

337     function deployAssets() internal { 
    uint newTickIndex = getActiveTickIndex();
    uint availToken0 = token0.balanceOf(address(this));
    uint availToken1 = token1.balanceOf(address(this));

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L337-L341

```solidity
file:

367    function poolMatchesOracle() public view returns (bool matches){
    (uint160 sqrtPriceX96,,,,,,) = uniswapPool.slot0();
    
    uint decimals0 = token0.decimals();
    uint decimals1 = token1.decimals();

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L367-L371

```solidity
file:

428     function getActiveTickIndex() public view returns (uint activeTickIndex) {
    if (ticks.length >= 5){
      // looking for index at which the underlying asset differs from the next tick
      for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){
        (uint amt0, uint amt1) = ticks[activeTickIndex+1].getTokenAmountsExcludingFees(1e18);
        (uint amt0n, uint amt1n) = ticks[activeTickIndex+2].getTokenAmountsExcludingFees(1e18);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L428-L433

```solidity
file:

445     function getAdjustedBaseFee(bool increaseToken0) public view returns (uint adjustedBaseFeeX4) {
    (uint res0, uint res1) = getReserves();
    uint value0 = res0 * oracle.getAssetPrice(address(token0)) / 10**token0.decimals();
    uint value1 = res1 * oracle.getAssetPrice(address(token1)) / 10**token1.decimals();

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L445-L448

```solidity
file:  contracts/RangeManager.sol

95     function initRange(address tr, uint amount0, uint amount1) external onlyOwner {
    ASSET_0.safeTransferFrom(msg.sender, address(this), amount0);
    ASSET_0.safeIncreaseAllowance(tr, amount0);
    ASSET_1.safeTransferFrom(msg.sender, address(this), amount1);
    ASSET_1.safeIncreaseAllowance(tr, amount1);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L95-L99

```solidity
file:  main/contracts/TokenisableRange.sol
333    function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {
    // if 0 get price from oracle
    if (TOKEN0_PRICE == 0) TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token));
    if (TOKEN1_PRICE == 0) TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token));


```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333-L336

```solidity
file:  contracts/helper/V3Proxy.sol

112       function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {
        require(path.length == 2, "Direct swap only");
        ERC20 ogInAsset = ERC20(path[0]);
        ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
        ogInAsset.safeApprove(address(ROUTER), amountIn);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L112-L116

```solidity
file:  contracts/helper/V3Proxy.sol
178    
    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
        require(path.length == 2, "Direct swap only");
        require(path[1] == ROUTER.WETH9(), "Invalid path");
        ERC20 ogInAsset = ERC20(path[0]);
        ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);
        ogInAsset.safeApprove(address(ROUTER), amountIn);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L178-L183


```solidity
file:  contracts/PositionManager/OptionsPositionManager.sol

250     function closeDebt(
    uint poolId, 
    address user,
    address debtAsset, 
    uint repayAmount,
    address collateralAsset
  ) 
    internal returns (uint debt)
  {
    (ILendingPool LP,,IUniswapV2Router01 ammRouter, address token0, address token1) = getPoolAddresses(poolId);
    sanityCheckUnderlying(debtAsset, token0, token1);
    require(collateralAsset == token0 || collateralAsset == token1, "OPM: Invalid Collateral Asset");
    uint amtA;
    uint amtB;
    // Add dust to be sure debt reformed >= debt outstanding
    debt = repayAmount + addDust(debtAsset, token0, token1);
    
    // Claim fees first so that deposit will match exactly
    TokenisableRange(debtAsset).claimFee();
    { //localize vars
      (uint token0Amount, uint token1Amount) = TokenisableRange(debtAsset).getTokenAmounts(debt);
      checkExpectedBalances(debtAsset, debt, token0Amount, token1Amount);
      checkSetAllowance(token0, debtAsset, token0Amount);
      checkSetAllowance(token1, debtAsset, token1Amount);
      // If called by this contract himself this is a liquidation, skip that step
      if (user != address(this) ){
        amtA = IERC20(LP.getReserveData(token0).aTokenAddress ).balanceOf(user);   // <= FOUND
        amtB = IERC20(LP.getReserveData(token1).aTokenAddress ).balanceOf(user);   // <= FOUND
        PMWithdraw(LP, user, token0, amtA );
        PMWithdraw(LP, user, token1, amtB );
        // If another user softLiquidates a share of the liquidation goes to the treasury
        if (user != msg.sender ) {
          uint feeAmount = calculateAndSendFee(poolId, token0Amount, token1Amount, collateralAsset);
          if (collateralAsset == token0) amtA -= feeAmount;
          else amtB -= feeAmount;
        }
      }
      else {
        // Assets are already present from liquidation
        amtA = ERC20(token0).balanceOf(user);    // <= FOUND
        amtB = ERC20(token1).balanceOf(user);   // <= FOUND
      }

      // swap if one token is missing - consider that there is enough 
      address[] memory path = new address[](2);
      if ( amtA < token0Amount ){
        path[0] = token1;
        path[1] = token0;
        swapTokensForExactTokens(ammRouter, token0Amount - amtA, amtB, path); 
      }
      else if ( amtB < token1Amount ){
        path[0] = token0;
        path[1] = token1;
        swapTokensForExactTokens(ammRouter, token1Amount - amtB, amtA, path); 
      }
      debt = TokenisableRange(debtAsset).deposit(token0Amount, token1Amount);
    }
    checkSetAllowance(debtAsset, address(LP), debt);
    
    // If user closes, repay debt, else tokens will be taken back by the flashloan
    if (user != address(this) ) LP.repay( debtAsset, debt, 2, user);
    {
      uint amt0 = ERC20(token0).balanceOf(address(this));   // <= FOUND
      uint amt1 = ERC20(token1).balanceOf(address(this));   // <= FOUND
      // edge case where after swapping exactly the tokens and repaying debt, dust causes remaining asset balance to be slightly higher than before repaying
      if (amtA > amt0) 
        amt0 = amtA - amt0;
      else 
        amt0 = 0;
      if (amtB > amt1) 
        amt1 = amtB - amt1;
      else 
        amt1 = 0;
      emit ClosePosition(user, debtAsset, debt, amt0, amt1);
    }
    
    // Swap other token back to collateral: this allows to control exposure
    if (user == msg.sender) swapTokens(poolId, collateralAsset == token0 ? token1 : token0, 0);
  }
  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L250-L328

## [G-05] Use calldata instead of memory for function arguments that do not get mutated

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.



```solidity
file:  contracts/PositionManager/OptionsPositionManager.sol

159     function buyOptions(
    uint poolId, 
    address[] memory options, 
    uint[] memory amounts, 
    address[] memory sourceSwap
  )
    external
  {

189    function liquidate (
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

## [G-06] Use custom errors instead of require/assert

Consider the use of a custom error, as it leads to a cheaper deploy cost and run time cost. The run time cost is only relevant when the revert condition is met.


```solidity
file: contracts/GeVault.sol

79     require(_treasury != address(0x0), "GEV: Invalid Treasury");

80     require(_uniswapPool != address(0x0), "GEV: Invalid Pool");

81     require(weth != address(0x0), "GEV: Invalid WETH");

120    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

125    require( t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap");

127    require( t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap");

141    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

146    require( t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");

148    require( t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");

170    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

184    require(newBaseFeeX4 < 1e4, "GEV: Invalid Base Fee");

203    require(poolMatchesOracle(), "GEV: Oracle Error");

215    require(poolMatchesOracle(), "GEV: Oracle Error");

217    require(liquidity <= balanceOf(msg.sender), "GEV: Insufficient Balance");

218    require(liquidity > 0, "GEV: Withdraw Zero");

249    require(isEnabled, "GEV: Pool Disabled");

250    require(poolMatchesOracle(), "GEV: Oracle Error");

251    require(token == address(token0) || token == address(token1), "GEV: Invalid Token");

252    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");

256    require(token == address(WETH), "GEV: Invalid Weth");

269    require(tvlCap > valueX8 + getTVL(), "GEV: Max Cap Reached")

281    require(liquidity > 0, "GEV: No Liquidity Added");
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L79

```solidity
file: contracts/RangeManager.sol

49    require( address(lendingPool) != address(0x0), "Invalid address" );

60    require(start < end, "Range invalid");

76    require(beacon != address(0x0), "Invalid beacon");

108   require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");

206   require(hf > 1.01e18, "Health factor is too low");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L49


```solidity
file:  main/contracts/RoeRouter.sol

34    require(treasury_ != address(0x0), "Invalid address");

68       require (
      lendingPoolAddressProvider != address(0x0) 
      && token0 != address(0x0) 
      && token1 != address(0x0) 
      && ammRouter != address(0x0), 
      "Invalid Address"
    );

75  require(token0 < token1, "Invalid Order");

84  require(newTreasury != address(0x0), "Invalid address");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L34

```solidity
file:  contracts/TokenisableRange.sol

86    require(address(_oracle) != address(0x0), "Invalid oracle");

87    require(status == ProxyState.INIT_PROXY, "!InitProxy");

135   require(status == ProxyState.INIT_LP, "!InitLP");

136   require(msg.sender == creator, "Unallowed call");

209   require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

225   require(totalSupply() > 0, "TR Closed"); 

271   require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L86

```solidity
file:  contracts/helper/FixedOracle.sol

11    require(msg.sender == owner, "Only the owner can call this function.");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11

```solidity
file:  contracts/helper/LPOracle.sol

29   require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");

63   require(timeStamp > 0, "Round not complete");

101  require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L29

```solidity
file:  contracts/helper/OracleConvert.sol

23    require(clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");

26    require(CL_TOKENA.decimals() + CL_TOKENB.decimals() >= 16, "Decimals error");

44    require(timeStamp > 0, "Round not complete");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L23

```solidity
file:  contracts/helper/V3Proxy.sol

87     require(acceptPayable, "CannotReceiveETH");

91     require(acceptPayable, "CannotReceiveETH");

99     require(path.length == 2, "Direct swap only");

106    require(path.length == 2, "Direct swap only");

113    require(path.length == 2, "Direct swap only");

125    require(path.length == 2, "Direct swap only");

138    require(path.length == 2, "Direct swap only");

148    require(path.length == 2, "Direct swap only");

161    require(path.length == 2, "Direct swap only");

179    require(path.length == 2, "Direct swap only");

180    require(path[1] == ROUTER.WETH9(), "Invalid path");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L87

```solidity
file:  contracts/PositionManager/OptionsPositionManager.sol

69    require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");

91    require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");

137   require(sourceSwap == token0 || sourceSwap == token1, "OPM: Invalid Swap Token");

167   require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");

198   require(options.length == amounts.length, "ARRAY_LEN_MISMATCH");

235   require(debt > 0, "OPM: No Debt");

261   require(collateralAsset == token0 || collateralAsset == token1, "OPM: Invalid Collateral Asset");

369   require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral");

393   require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

420   require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

441   require(sourceAsset == token0 || sourceAsset == token1, "OPM: Invalid Swap Asset");

495   require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );

496   require( amountsIn[0] <= ERC20(path[0]).balanceOf(address(this)), "OPM: Insufficient Token Amount" );

523   require ( priceAssetA > 0 && priceAssetB > 0, "OPM: Invalid Oracle Price");

525   require( amountB > 0, "OPM: Target Amount Too Low");

536   require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L69

```solidity
file: contracts/PositionManager/PositionManager.sol

30   require(roerouter != address(0x0), "Invalid address");

145  require( valueA <= valueB * 101 / 100, "PM: LP Oracle Error");

146  require( valueB <= valueA * 101 / 100, "PM: LP Oracle Error");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L30

## [G-07] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking thefunction as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a paymentwas provided.



```solidity
file:  contracts/GeVault.sol

101  function setEnabled(bool _isEnabled) public onlyOwner { 

018  function setTreasury(address newTreasury) public onlyOwner { 

116  function pushTick(address tr) public onlyOwner {

137  function shiftTick(address tr) public onlyOwner {

167  function modifyTick(address tr, uint index) public onlyOwner {

183  function setBaseFee(uint newBaseFeeX4) public onlyOwner {

191  function setTvlCap(uint newTvlCap) public onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101

```solidity
file:  contracts/RangeManager.sol

75   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

95   function initRange(address tr, uint amount0, uint amount1) external onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75

```solidity
file: contracts/RoeRouter.sol

48   function deprecatePool(uint poolId) public onlyOwner {

59    function addPool(
    address lendingPoolAddressProvider, 
    address token0, 
    address token1, 
    address ammRouter
  ) 
    public onlyOwner 

83   function setTreasury(address newTreasury) public onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L48

```solidity
file: contracts/helper/FixedOracle.sol

24     function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24

```solidity
file:  contracts/helper/V3Proxy.sol

94    function emergencyWithdraw(ERC20 token) onlyOwner external {  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L94

## [G-08] Use assembly to emit events

We can use assembly to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs. Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.


```solidity
file:  contracts/GeVault.sol

103  emit SetEnabled(_isEnabled);

110  emit SetTreasury(newTreasury);

131  emit PushTick(tr);

160  emit ShiftTick(tr);

172  emit ModifyTick(tr, index);

186  emit SetFee(newBaseFeeX4);

193  emit SetTvlCap(newTvlCap);

240  emit Withdraw(msg.sender, token, amount, liquidity);

283  emit Deposit(msg.sender, token, amount, liquidity);

362  emit Rebalance(tickIndex);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L103

```solidity
file: contracts/RangeManager.sol

87    emit AddRange(startX10, endX10, tokenisedRanges.length - 1);

120   emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);

132   emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);

164   emit Deposit(msg.sender, address(tr), lpAmt);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L87

```solidity
file: contracts/RoeRouter.sol

50   emit DeprecatePool(poolId);

78   emit AddPool(poolId, lendingPoolAddressProvider);

86   emit UpdateTreasury(newTreasury);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L50

```solidity
file: contracts/TokenisableRange.sol

119   emit InitTR(address(asset0), address(asset1), startX10, endX10);

162   emit Deposit(msg.sender, 1e18);

214   emit ClaimFees(newFee0, newFee1);

284   emit Deposit(msg.sender, lpAmt);

326   emit Withdraw(msg.sender, lp);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L119

```solidity
file: contracts/helper/V3Proxy.sol

121   emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 

134   emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 

143   emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 

157   emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 

175   emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 

193   emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]); 

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L121

```solidity
file: contracts/PositionManager/OptionsPositionManager.sol

106   emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1]);

148   emit BuyOptions(user, flashAsset, flashAmount, amount0, amount1);

240   emit ReducedPosition(user, debtAsset, debt);

323   emit ClosePosition(user, debtAsset, debt, amt0, amt1);

401   emit SellOptions(msg.sender, optionAddress, deposited, amount0, amount1 );

455   emit Swap(msg.sender, path[0], amount, path[1], received);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L106

## [G-09] Use assembly to write address storage values

```solidity
file:  contracts/GeVault.sol

108     function setTreasury(address newTreasury) public onlyOwner { 
    treasury = newTreasury; 
    emit SetTreasury(newTreasury);
  }

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L108-111

```solidity
file:  contracts/RoeRouter.sol

83     function setTreasury(address newTreasury) public onlyOwner {
    require(newTreasury != address(0x0), "Invalid address");
    treasury = newTreasury;
    emit UpdateTreasury(newTreasury);
  }

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L83-L87


## [G-10] Use hardcoded address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's script.sol and solmate's LibRlp.sol contracts can help achieve this. Refrences

```solidity
file: contracts/GeVault.sol

262   ERC20(token).safeTransferFrom(msg.sender, address(this), amount);

302   uint bal = ERC20(aTick).balanceOf(address(this));

324   uint aBal = ERC20(aTokenAddress).balanceOf(address(this));

330   lendingPool.withdraw(address(tr), aBal, address(this));

339   uint availToken0 = token0.balanceOf(address(this));

340   uint availToken1 = token1.balanceOf(address(this));

386   if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

409   uint bal = t.balanceOf(address(this));

412   lendingPool.deposit(address(t), bal, address(this), 0);

423    liquidity = ERC20(aTokenAddress).balanceOf(address(this));

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L262

```solidity
file:  contracts/RangeManager.sol
 
96    ASSET_0.safeTransferFrom(msg.sender, address(this), amount0);

98    ASSET_1.safeTransferFrom(msg.sender, address(this), amount1);

101   ERC20(tr).safeTransfer(msg.sender, TokenisableRange(tr).balanceOf(address(this)));

155   LENDING_POOL.withdraw( address(ASSET_0), amount0, address(this) );

160   LENDING_POOL.withdraw( address(ASSET_1), amount1, address(this) );

191   uint256 asset0_amt = ASSET_0.balanceOf(address(this));

192   uint256 asset1_amt = ASSET_1.balanceOf(address(this));

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L96

```solidity
file:  contracts/TokenisableRange.sol

138    TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);

139    TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);

159    TOKEN0.token.safeTransfer( msg.sender,  TOKEN0.token.balanceOf(address(this)));

160    TOKEN1.token.safeTransfer(msg.sender, TOKEN1.token.balanceOf(address(this)));

170    recipient: address(this),

228    TOKEN0.token.transferFrom(msg.sender, address(this), n0);

229    TOKEN1.token.transferFrom(msg.sender, address(this), n1);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138

```solidity
file: contracts/helper/V3Proxy.sol

95   token.safeTransfer(msg.sender, token.balanceOf( address(this) ) ); 

115  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);

127  ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);

132  ogInAsset.safeTransfer(msg.sender, ogInAsset.balanceOf(address(this)));

164  ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);

182  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L95

```solidity
file:  contracts/PositionManager/OptionsPositionManager.sol

92   uint[2] memory amts = [ERC20(token0).balanceOf(address(this)), ERC20(token1).balanceOf(address(this))];

103   uint debt = closeDebt(poolId, address(this), debtAsset, amount, collateral);

104   uint amt0 = ERC20(token0).balanceOf(address(this));

105   uint amt1 = ERC20(token1).balanceOf(address(this));

176   LP.flashLoan( address(this), options, amounts, flashtype, msg.sender, params, 0);

207   LP.flashLoan( address(this), options, amounts, flashtype, msg.sender, params, 0);

275   if (user != address(this) ){

312   uint amt0 = ERC20(token0).balanceOf(address(this));

313   uint amt1 = ERC20(token1).balanceOf(address(this));

369   require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral");

479   address(this), 

496   require( amountsIn[0] <= ERC20(path[0]).balanceOf(address(this)), "OPM: Insufficient Token Amount" );

501   address(this),

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L103

```solidity
file:  contracts/PositionManager/PositionManager.sol

81    if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

90    uint amt = ERC20(asset).balanceOf(address(this));

115   ERC20(asset).safeTransfer(ROEROUTER.treasury(), ERC20(asset).balanceOf(address(this))); 

127   LP.withdraw(asset, amount, address(this));

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L81

## [G-11] Use uint256(1)/uint256(2) instead for true and false boolean states

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.
see source:  
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27


```solidity
file: contracts/GeVault.sol

50    bool public isEnabled = true;

64    bool public baseTokenIsToken0;

75    bool _baseTokenIsToken0

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L50

```solidity
file: contracts/RoeRouter.sol

28    bool isDeprecated;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L28

```solidity
file:  contracts/helper/V3Proxy.sol

65   bool acceptPayable;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L65

```solidity
file: contracts/interfaces/IAaveLendingPoolV2.sol

146   bool receiveAToken

288   bool receiveAToken

235   bool _state

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/interfaces/IAaveLendingPoolV2.sol#L146

## [G-12] Expensive operation inside a for loop

```solidity
file:  contracts/RangeManager.sol

62      for (uint i = 0; i < len; i++) {
      if (start >= stepList[i].end || end <= stepList[i].start) {
        continue;
      }
      revert("Range overlap");
    } 

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L62-L67

## [G-13] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.


```solidity
file:  contracts/PositionManager/OptionsPositionManager.sol

281    if (user != msg.sender ) {

327    if (user == msg.sender) swapTokens(poolId, collateralAsset == token0 ? token1 : token0, 0);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L281

```solidity
file:  contracts/TokenisableRange.sol

136    require(msg.sender == creator, "Unallowed call");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L136

```solidity
file:  contracts/helper/FixedOracle.sol

11   require(msg.sender == owner, "Only the owner can call this function.");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11

```solidity
file:  contracts/PositionManager/OptionsPositionManager.sol

91    require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L91

## [N-14] Duplicated require()/revert() Checks Should Be Refactored To A Modifier Or Function

In development process, if you changed one of them you should find all of other to change and for large and complicatedprojects possible this change will be missed.

```solidity
file:  contracts/GeVault.sol
   
   /// @audit this require is duplicated on line 215, 250

203   require(poolMatchesOracle(), "GEV: Oracle Error");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L203

```solidity
file: contracts/helper/V3Proxy.sol

   /// @audit this require is duplicated on line 91

87   require(acceptPayable, "CannotReceiveETH");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L87

```solidity
file:
 
 /// @audit this require is duplicated on line 106, 113, 125, 138, 148, 161, 179

99   require(path.length == 2, "Direct swap only");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L99

## [G-15] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file:  contracts/helper/FixedOracle.sol

10  modifier onlyOwner() {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L10

## [G-16] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

Note that in order to do this optimization safely we will need to cache and restore the free memory pointer after the loop. We will also set the zero slot (0x60) back to 0.


```solidity
file:  contracts/GeVault.sol

314       for (uint k = 0; k < ticks.length; k++){
      removeFromTick(k);
    } 

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L314-L316


## [G‑17] require()/revert() strings longer than 32 bytes cost extra gas

```solidity
file: contracts/helper/FixedOracle.sol  

11   require(msg.sender == owner, "Only the owner can call this function.");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11

```solidity
file:  contracts/PositionManager/OptionsPositionManager.sol

495   require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );

496   require( amountsIn[0] <= ERC20(path[0]).balanceOf(address(this)), "OPM: Insufficient Token Amount" );

525   require( amountB > 0, "OPM: Target Amount Too Low");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L495

## [G‑18] Using private rather than public for constants, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.

saved gas: 50436

```solidity
file: contracts/GeVault.sol

62   uint public constant nearbyRanges = 2;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L62

```solidity 
file: contracts/RangeManager.sol

36   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36

```solidity
file:  contracts/TokenisableRange.sol

58   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88); 

59   IUniswapV3Factory constant public V3_FACTORY = IUniswapV3Factory(0x1F98431c8aD98523631AE4a59f267346ea31F984); 

60   address constant public treasury = 0x22Cc3f665ba4C898226353B672c5123c58751692;

61   uint constant public treasuryFee = 20;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L58

## [G-19] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value 


```solidity
file:  contracts/RangeManager.sol

96     ASSET_0.safeTransferFrom(msg.sender, address(this), amount0);

98     ASSET_1.safeTransferFrom(msg.sender, address(this), amount1);

176    transferAssetsIntoStep(tokenisedRanges[step], step, amount0, amount1);

185    transferAssetsIntoStep(tokenisedTicker[step], step, amount0, amount1);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L96


```solidity
file:  contracts/helper/V3Proxy.sol

164    ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);

165    ogInAsset.safeApprove(address(ROUTER), amountInMax);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L164


## [G-20] Use constants instead of type(uintx).max

type(uint120).max or type(uint112).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file:   contracts/GeVault.sol

386   if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L386

```solidity
file: contracts/RangeManager.sol

118   trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));

13    uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L118

```solidity
file: contracts/TokenisableRange.sol

172   amount0Max: type(uint128).max,

173   amount1Max: type(uint128).max,

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L172

```solidity
file:  contracts/helper/OracleConvert.sol

79    return (type(uint80).max, latestAnswer(), block.timestamp, block.timestamp, type(uint80).max);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L79


```solidity
file:  contracts/PositionManager/PositionManager.sol

81     if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L81


## [G-21] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas  

```solidity
file: /contracts/GeVault.sol

362    emit Rebalance(tickIndex);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362

## [G-22] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity
file: /contracts/TokenisableRange.sol

88    creator = msg.sender;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L88


## [G-23] Empty blocks should be removed or emit something

f the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be 
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when 
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) . 
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

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

## [G-24] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

72      address asset = assets[k];

73      uint amount = amounts[k];


94      address debtAsset = assets[k];

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L72-L73


```solidity
file: /contracts/GeVault.sol

301      address aTick = lendingPool.getReserveData(address(t)).aTokenAddress;

302      uint bal = ERC20(aTick).balanceOf(address(this));

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L301-L302

## [G-25]  abi.encode() is less efficient than  abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

168    bytes memory params = abi.encode(0, poolId, msg.sender, sourceSwap);

199    bytes memory params = abi.encode(1, poolId, user, collateralAsset); // mode = 1 -> liquidation

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L168

## [G-26] Avoid contract existence checks by using low level calls		

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.					

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

## [G-27] Using delete statement can save gas

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

204      flashtype[i] = 0; // dont open debt for liquidations, need to repay

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L204

## [G-28] Not using the named return variable when a function returns, wastes deployment gas

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


## [G-29] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

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

