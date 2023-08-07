# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 218 at 2023-08-good-entry/contracts/GeVault.sol:

    require(liquidity > 0, "GEV: Withdraw Zero");


Found in line 252 at 2023-08-good-entry/contracts/GeVault.sol:

    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");


Found in line 255 at 2023-08-good-entry/contracts/GeVault.sol:

    if (msg.value > 0){


Found in line 281 at 2023-08-good-entry/contracts/GeVault.sol:

    require(liquidity > 0, "GEV: No Liquidity Added");


Found in line 329 at 2023-08-good-entry/contracts/GeVault.sol:

    if (aBal > 0){


Found in line 346 at 2023-08-good-entry/contracts/GeVault.sol:

    if (amount1ft > 0){


Found in line 352 at 2023-08-good-entry/contracts/GeVault.sol:

    if (availToken0 > 0){


Found in line 356 at 2023-08-good-entry/contracts/GeVault.sol:

    if (availToken1 > 0){


Found in line 410 at 2023-08-good-entry/contracts/GeVault.sol:

    if (bal > 0){


Found in line 434 at 2023-08-good-entry/contracts/GeVault.sol:

        if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )


Found in line 180 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    if (tf0 > 0) TOKEN0.token.safeTransfer(treasury, tf0);


Found in line 181 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    if (tf1 > 0) TOKEN1.token.safeTransfer(treasury, tf1);


Found in line 225 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    require(totalSupply() > 0, "TR Closed"); 


Found in line 237 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) ){


Found in line 241 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      if (token0Amount + fee0 > 0) newFee0 = n0 * fee0 / (token0Amount + fee0);


Found in line 242 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      if (token1Amount + fee1 > 0) newFee1 = n1 * fee1 / (token1Amount + fee1);


Found in line 271 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");


Found in line 315 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    if (fee0 > 0) {


Found in line 320 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    if (fee1 > 0) {


Found in line 112 at 2023-08-good-entry/contracts/RangeManager.sol:

    if (trAmt > 0) {       


Found in line 124 at 2023-08-good-entry/contracts/RangeManager.sol:

    if (trAmt > 0) {    


Found in line 153 at 2023-08-good-entry/contracts/RangeManager.sol:

    if (amount0 > 0) {    


Found in line 158 at 2023-08-good-entry/contracts/RangeManager.sol:

    if (amount1 > 0) {


Found in line 194 at 2023-08-good-entry/contracts/RangeManager.sol:

    if (asset0_amt > 0) {


Found in line 199 at 2023-08-good-entry/contracts/RangeManager.sol:

    if (asset1_amt > 0) {


Found in line 63 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    require(timeStamp > 0, "Round not complete");


Found in line 44 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    require(timeStamp > 0, "Round not complete");


Found in line 234 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    if ( repayAmount > 0 && repayAmount < debt ) debt = repayAmount;


Found in line 235 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(debt > 0, "OPM: No Debt");


Found in line 473 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    if (amount > 0 && AmountsRouter(address(ammRouter)).getAmountsOut(amount, path)[1] > 0){


Found in line 495 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );


Found in line 523 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require ( priceAssetA > 0 && priceAssetB > 0, "OPM: Invalid Oracle Price");


Found in line 525 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( amountB > 0, "OPM: Target Amount Too Low");


Found in line 91 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    if (amt > 0) {


Found in line 96 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

      if ( debt > 0 ){


Found in line 125 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    if ( amount > 0 ){

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] Use shift Right/Left instead of division/multiplication if possible
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 268 at 2023-08-good-entry/contracts/GeVault.sol:

    uint valueX8 = oracle.getAssetPrice(token) * (amount - fee) / 10**ERC20(token).decimals();


Found in line 353 at 2023-08-good-entry/contracts/GeVault.sol:

      depositAndStash(ticks[tick0Index], availToken0 / 2, 0);


Found in line 354 at 2023-08-good-entry/contracts/GeVault.sol:

      depositAndStash(ticks[tick0Index+1], availToken0 / 2, 0);


Found in line 357 at 2023-08-good-entry/contracts/GeVault.sol:

      depositAndStash(ticks[tick1Index], 0, availToken1 / 2);


Found in line 358 at 2023-08-good-entry/contracts/GeVault.sol:

      depositAndStash(ticks[tick1Index+1], 0, availToken1 / 2);


Found in line 374 at 2023-08-good-entry/contracts/GeVault.sol:

    priceX8 = priceX8 * uint(sqrtPriceX96 / 2 ** 12) ** 2 * 1e8 / 2**168;


Found in line 375 at 2023-08-good-entry/contracts/GeVault.sol:

    priceX8 = priceX8 / 10**decimals1;


Found in line 377 at 2023-08-good-entry/contracts/GeVault.sol:

    if (oraclePrice < priceX8 * 101 / 100 && oraclePrice > priceX8 * 99 / 100) matches = true;


Found in line 447 at 2023-08-good-entry/contracts/GeVault.sol:

    uint value0 = res0 * oracle.getAssetPrice(address(token0)) / 10**token0.decimals();


Found in line 448 at 2023-08-good-entry/contracts/GeVault.sol:

    uint value1 = res1 * oracle.getAssetPrice(address(token1)) / 10**token1.decimals();


Found in line 456 at 2023-08-good-entry/contracts/GeVault.sol:

    if (adjustedBaseFeeX4 < baseFeeX4 / 2) adjustedBaseFeeX4 = baseFeeX4 / 2;


Found in line 457 at 2023-08-good-entry/contracts/GeVault.sol:

    if (adjustedBaseFeeX4 > baseFeeX4 * 3 / 2) adjustedBaseFeeX4 = baseFeeX4 * 3 / 2;


Found in line 67 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      uint z = (x + 1) / 2;


Found in line 71 at 2023-08-good-entry/contracts/TokenisableRange.sol:

          z = (x / z + z) / 2;


Found in line 105 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      midleTick = (_upperTick + _lowerTick) / 2;


Found in line 151 at 2023-08-good-entry/contracts/TokenisableRange.sol:

         amount0Min: n0 * 95 / 100,


Found in line 152 at 2023-08-good-entry/contracts/TokenisableRange.sol:

         amount1Min: n1 * 95 / 100,


Found in line 178 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    uint tf0 = newFee0 * treasuryFee / 100;


Found in line 179 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    uint tf1 = newFee1 * treasuryFee / 100;


Found in line 206 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      uint addedValue = added0 * token0Price / 10**TOKEN0.decimals + added1 * token1Price / 10**TOKEN1.decimals;


Found in line 207 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      uint totalValue =   bal0 * token0Price / 10**TOKEN0.decimals +   bal1 * token1Price / 10**TOKEN1.decimals;


Found in line 209 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");


Found in line 258 at 2023-08-good-entry/contracts/TokenisableRange.sol:

        amount0Min: n0 * 95 / 100,


Found in line 259 at 2023-08-good-entry/contracts/TokenisableRange.sol:

        amount1Min: n1 * 95 / 100,


Found in line 274 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      feeLiquidity = newLiquidity * ( (fee0 * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (fee1 * TOKEN1_PRICE / 10 ** TOKEN1.decimals) )   


Found in line 275 at 2023-08-good-entry/contracts/TokenisableRange.sol:

                                    / ( (added0   * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (added1   * TOKEN1_PRICE / 10 ** TOKEN1.decimals) ); 


Found in line 45 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    uint z = (x + 1) / 2;


Found in line 49 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

      z = (x / z + z) / 2;


Found in line 92 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

      norm_b = sqrt( a * b * priceA / 10**(decimalsA-decimalsB) / priceB );


Found in line 342 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    uint tokensValue = token0Amount * oracle.getAssetPrice(address(token0)) / 10**decimals0 + token1Amount * oracle.getAssetPrice(address(token1)) / 10**decimals1;


Found in line 346 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      || (tokensValue > debtValue * 98 / 100 && tokensValue < debtValue * 102 / 100), 


Found in line 365 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    uint feeValueE8 = token0Amount * oracle.getAssetPrice(token0) / 10**ERC20(token0).decimals()


Found in line 366 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

                    + token1Amount * oracle.getAssetPrice(token1) / 10**ERC20(token1).decimals() ;


Found in line 367 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    feeAmount = feeValueE8 * 10**ERC20(collateralAsset).decimals() / 100 / oracle.getAssetPrice(collateralAsset);


Found in line 477 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

        getTargetAmountFromOracle(oracle, path[0], amount, path[1]) * 99 / 100, // allow 1% slippage 


Found in line 517 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      uint valueA = amountA * oracle.getAssetPrice(assetA) / 10**ERC20(assetA).decimals();


Found in line 518 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      uint valueB = amountB * oracle.getAssetPrice(assetB) / 10**ERC20(assetB).decimals();


Found in line 524 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    amountB = amountA * priceAssetA * 10**ERC20(assetB).decimals() / 10**ERC20(assetA).decimals() / priceAssetB;


Found in line 143 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    if (decimalsA > decimalsB) valueA = valueA / 10 ** (decimalsA - decimalsB);


Found in line 144 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    else if (decimalsA < decimalsB) valueB = valueB / 10 ** (decimalsB - decimalsA);


Found in line 145 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    require( valueA <= valueB * 101 / 100, "PM: LP Oracle Error");


Found in line 146 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    require( valueB <= valueA * 101 / 100, "PM: LP Oracle Error");

------------------------------------------------------------------------ 

### Mitigation 

Use shift Right/Left instead of division/multiplication if possible.










# VULN 3 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 154 at 2023-08-good-entry/contracts/GeVault.sol:

        for (uint k = 0; k < ticks.length - 2; k++) 


Found in line 299 at 2023-08-good-entry/contracts/GeVault.sol:

    for (uint k = 0; k < ticks.length; k++){


Found in line 314 at 2023-08-good-entry/contracts/GeVault.sol:

    for (uint k = 0; k < ticks.length; k++){


Found in line 393 at 2023-08-good-entry/contracts/GeVault.sol:

    for(uint k=0; k<ticks.length; k++){


Found in line 431 at 2023-08-good-entry/contracts/GeVault.sol:

      for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){


Found in line 62 at 2023-08-good-entry/contracts/RangeManager.sol:

    for (uint i = 0; i < len; i++) {


Found in line 71 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for ( uint8 k = 0; k<assets.length; k++){


Found in line 93 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for ( uint8 k =0; k<assets.length; k++){

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 4 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 154 at 2023-08-good-entry/contracts/GeVault.sol:

        for (uint k = 0; k < ticks.length - 2; k++) 


Found in line 299 at 2023-08-good-entry/contracts/GeVault.sol:

    for (uint k = 0; k < ticks.length; k++){


Found in line 314 at 2023-08-good-entry/contracts/GeVault.sol:

    for (uint k = 0; k < ticks.length; k++){


Found in line 62 at 2023-08-good-entry/contracts/RangeManager.sol:

    for (uint i = 0; i < len; i++) {


Found in line 71 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for ( uint8 k = 0; k<assets.length; k++){


Found in line 172 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for (uint8 i = 0; i< options.length; ){


Found in line 203 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for (uint8 i = 0; i< options.length; ){

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 5 

## [GAS] Use Custom Errors
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 79 at 2023-08-good-entry/contracts/GeVault.sol:

    require(_treasury != address(0x0), "GEV: Invalid Treasury");


Found in line 80 at 2023-08-good-entry/contracts/GeVault.sol:

    require(_uniswapPool != address(0x0), "GEV: Invalid Pool");


Found in line 81 at 2023-08-good-entry/contracts/GeVault.sol:

    require(weth != address(0x0), "GEV: Invalid WETH");


Found in line 120 at 2023-08-good-entry/contracts/GeVault.sol:

    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");


Found in line 125 at 2023-08-good-entry/contracts/GeVault.sol:

        require( t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap");


Found in line 127 at 2023-08-good-entry/contracts/GeVault.sol:

        require( t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap");


Found in line 141 at 2023-08-good-entry/contracts/GeVault.sol:

    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");


Found in line 146 at 2023-08-good-entry/contracts/GeVault.sol:

        require( t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");


Found in line 148 at 2023-08-good-entry/contracts/GeVault.sol:

        require( t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");


Found in line 170 at 2023-08-good-entry/contracts/GeVault.sol:

    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");


Found in line 184 at 2023-08-good-entry/contracts/GeVault.sol:

  require(newBaseFeeX4 < 1e4, "GEV: Invalid Base Fee");


Found in line 203 at 2023-08-good-entry/contracts/GeVault.sol:

    require(poolMatchesOracle(), "GEV: Oracle Error");


Found in line 215 at 2023-08-good-entry/contracts/GeVault.sol:

    require(poolMatchesOracle(), "GEV: Oracle Error");


Found in line 217 at 2023-08-good-entry/contracts/GeVault.sol:

    require(liquidity <= balanceOf(msg.sender), "GEV: Insufficient Balance");


Found in line 218 at 2023-08-good-entry/contracts/GeVault.sol:

    require(liquidity > 0, "GEV: Withdraw Zero");


Found in line 249 at 2023-08-good-entry/contracts/GeVault.sol:

    require(isEnabled, "GEV: Pool Disabled");


Found in line 250 at 2023-08-good-entry/contracts/GeVault.sol:

    require(poolMatchesOracle(), "GEV: Oracle Error");


Found in line 251 at 2023-08-good-entry/contracts/GeVault.sol:

    require(token == address(token0) || token == address(token1), "GEV: Invalid Token");


Found in line 252 at 2023-08-good-entry/contracts/GeVault.sol:

    require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");


Found in line 256 at 2023-08-good-entry/contracts/GeVault.sol:

      require(token == address(WETH), "GEV: Invalid Weth");


Found in line 269 at 2023-08-good-entry/contracts/GeVault.sol:

    require(tvlCap > valueX8 + getTVL(), "GEV: Max Cap Reached");


Found in line 281 at 2023-08-good-entry/contracts/GeVault.sol:

    require(liquidity > 0, "GEV: No Liquidity Added");


Found in line 34 at 2023-08-good-entry/contracts/RoeRouter.sol:

    require(treasury_ != address(0x0), "Invalid address");


Found in line 75 at 2023-08-good-entry/contracts/RoeRouter.sol:

    require(token0 < token1, "Invalid Order");


Found in line 84 at 2023-08-good-entry/contracts/RoeRouter.sol:

    require(newTreasury != address(0x0), "Invalid address");


Found in line 86 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    require(address(_oracle) != address(0x0), "Invalid oracle");


Found in line 87 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    require(status == ProxyState.INIT_PROXY, "!InitProxy");


Found in line 135 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    require(status == ProxyState.INIT_LP, "!InitLP");


Found in line 136 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    require(msg.sender == creator, "Unallowed call");


Found in line 209 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");


Found in line 225 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    require(totalSupply() > 0, "TR Closed"); 


Found in line 271 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");


Found in line 60 at 2023-08-good-entry/contracts/RangeManager.sol:

    require(start < end, "Range invalid");


Found in line 66 at 2023-08-good-entry/contracts/RangeManager.sol:

      revert("Range overlap");


Found in line 76 at 2023-08-good-entry/contracts/RangeManager.sol:

    require(beacon != address(0x0), "Invalid beacon");


Found in line 108 at 2023-08-good-entry/contracts/RangeManager.sol:

    require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");


Found in line 206 at 2023-08-good-entry/contracts/RangeManager.sol:

    require(hf > 1.01e18, "Health factor is too low");


Found in line 29 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");


Found in line 63 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    require(timeStamp > 0, "Round not complete");


Found in line 101 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");


Found in line 11 at 2023-08-good-entry/contracts/helper/FixedOracle.sol:

        require(msg.sender == owner, "Only the owner can call this function.");


Found in line 23 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    require(clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");


Found in line 26 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    require(CL_TOKENA.decimals() + CL_TOKENB.decimals() >= 16, "Decimals error");


Found in line 44 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    require(timeStamp > 0, "Round not complete");


Found in line 87 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(acceptPayable, "CannotReceiveETH");


Found in line 91 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

       require(acceptPayable, "CannotReceiveETH");


Found in line 99 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path.length == 2, "Direct swap only");


Found in line 106 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path.length == 2, "Direct swap only");


Found in line 113 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path.length == 2, "Direct swap only");


Found in line 125 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path.length == 2, "Direct swap only");


Found in line 138 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path.length == 2, "Direct swap only");


Found in line 139 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path[0] == ROUTER.WETH9(), "Invalid path");


Found in line 148 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path.length == 2, "Direct swap only");


Found in line 149 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path[0] == ROUTER.WETH9(), "Invalid path");


Found in line 161 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path.length == 2, "Direct swap only");


Found in line 162 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path[1] == ROUTER.WETH9(), "Invalid path");


Found in line 179 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path.length == 2, "Direct swap only");


Found in line 180 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        require(path[1] == ROUTER.WETH9(), "Invalid path");


Found in line 69 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");


Found in line 91 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");


Found in line 137 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      require(sourceSwap == token0 || sourceSwap == token1, "OPM: Invalid Swap Token");


Found in line 167 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");


Found in line 198 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(options.length == amounts.length, "ARRAY_LEN_MISMATCH");


Found in line 235 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(debt > 0, "OPM: No Debt");


Found in line 261 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(collateralAsset == token0 || collateralAsset == token1, "OPM: Invalid Collateral Asset");


Found in line 369 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral");


Found in line 441 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(sourceAsset == token0 || sourceAsset == token1, "OPM: Invalid Swap Asset");


Found in line 523 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require ( priceAssetA > 0 && priceAssetB > 0, "OPM: Invalid Oracle Price");


Found in line 525 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( amountB > 0, "OPM: Target Amount Too Low");


Found in line 536 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");


Found in line 30 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    require(roerouter != address(0x0), "Invalid address");


Found in line 145 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    require( valueA <= valueB * 101 / 100, "PM: LP Oracle Error");


Found in line 146 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    require( valueB <= valueA * 101 / 100, "PM: LP Oracle Error");

------------------------------------------------------------------------ 

### Mitigation 

Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.










# VULN 6 

## [GAS] Use calldata instead of memory for function arguments that do not get mutated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 85 at 2023-08-good-entry/contracts/TokenisableRange.sol:

  function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {


Found in line 75 at 2023-08-good-entry/contracts/RangeManager.sol:

  function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {


Found in line 470 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

  function swapExactTokensForTokens(IUniswapV2Router01 ammRouter, IPriceOracle oracle, uint amount, address[] memory path) 


Found in line 491 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

  function swapTokensForExactTokens(IUniswapV2Router01 ammRouter, uint recvAmount, uint maxAmount, address[] memory path) internal {

------------------------------------------------------------------------ 

### Mitigation 

Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.










# VULN 7 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 154 at 2023-08-good-entry/contracts/GeVault.sol:

        for (uint k = 0; k < ticks.length - 2; k++) 


Found in line 299 at 2023-08-good-entry/contracts/GeVault.sol:

    for (uint k = 0; k < ticks.length; k++){


Found in line 314 at 2023-08-good-entry/contracts/GeVault.sol:

    for (uint k = 0; k < ticks.length; k++){


Found in line 431 at 2023-08-good-entry/contracts/GeVault.sol:

      for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){


Found in line 71 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for ( uint8 k = 0; k<assets.length; k++){


Found in line 93 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for ( uint8 k =0; k<assets.length; k++){


Found in line 172 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for (uint8 i = 0; i< options.length; ){


Found in line 203 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for (uint8 i = 0; i< options.length; ){

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 8 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 30 at 2023-08-good-entry/contracts/TokenisableRange.sol:

  uint24 public feeTier;


Found in line 38 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    uint8 decimals;


Found in line 8 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

  function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);


Found in line 14 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

  function decimals() external view returns (uint8);


Found in line 22 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

	uint8 public immutable decimalsA;


Found in line 23 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

	uint8 public immutable decimalsB;


Found in line 38 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

  function decimals() external pure returns (uint8) {


Found in line 30 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

  function decimals() external pure returns (uint8) {


Found in line 13 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        uint24 fee;


Found in line 25 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        uint24 fee;


Found in line 42 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        uint24 fee,


Found in line 50 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        uint24 fee,


Found in line 64 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

    uint24      immutable public feeTier; 


Found in line 75 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

    constructor(ISwapRouter _router, IQuoter _quoter, uint24 _fee) {


Found in line 45 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    uint8 mode = abi.decode(params, (uint8) );


Found in line 48 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));


Found in line 53 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));


Found in line 71 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for ( uint8 k = 0; k<assets.length; k++){


Found in line 93 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for ( uint8 k =0; k<assets.length; k++){


Found in line 172 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for (uint8 i = 0; i< options.length; ){


Found in line 203 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    for (uint8 i = 0; i< options.length; ){


Found in line 339 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    (ERC20 token0, uint8 decimals0) = TokenisableRange(debtAsset).TOKEN0();


Found in line 340 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    (ERC20 token1, uint8 decimals1) = TokenisableRange(debtAsset).TOKEN1();

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 9 

## [GAS] Public Functions to external
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 125 at 2023-08-good-entry/contracts/TokenisableRange.sol:

  function name()     public view virtual override returns (string memory) { return _name; }


Found in line 127 at 2023-08-good-entry/contracts/TokenisableRange.sol:

  function symbol()   public view virtual override returns (string memory) { return _symbol; }

------------------------------------------------------------------------ 

### Mitigation 

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.










# VULN 10 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 304 at 2023-08-good-entry/contracts/GeVault.sol:

      amount0 += amt0;


Found in line 305 at 2023-08-good-entry/contracts/GeVault.sol:

      amount1 += amt1;


Found in line 396 at 2023-08-good-entry/contracts/GeVault.sol:

      valueX8 += bal * t.latestAnswer() / 1e18;


Found in line 243 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      fee0 += newFee0;


Found in line 244 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      fee1 += newFee1; 


Found in line 317 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      removed0 += fee0 * lp / totalSupply();


Found in line 322 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      removed1 += fee1 * lp / totalSupply();


Found in line 345 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    amt0 += fee0;


Found in line 346 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    amt1 += fee1;


Found in line 379 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    token0Amount += fee0 * amount / totalSupply();


Found in line 380 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    token1Amount += fee1 * amount / totalSupply();


Found in line 174 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      unchecked { i+=1; }


Found in line 205 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      unchecked { i+=1; }

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 11 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 262 at 2023-08-good-entry/contracts/GeVault.sol:

      ERC20(token).safeTransferFrom(msg.sender, address(this), amount);


Found in line 302 at 2023-08-good-entry/contracts/GeVault.sol:

      uint bal = ERC20(aTick).balanceOf(address(this));


Found in line 324 at 2023-08-good-entry/contracts/GeVault.sol:

    uint aBal = ERC20(aTokenAddress).balanceOf(address(this));


Found in line 330 at 2023-08-good-entry/contracts/GeVault.sol:

      lendingPool.withdraw(address(tr), aBal, address(this));


Found in line 339 at 2023-08-good-entry/contracts/GeVault.sol:

    uint availToken0 = token0.balanceOf(address(this));


Found in line 340 at 2023-08-good-entry/contracts/GeVault.sol:

    uint availToken1 = token1.balanceOf(address(this));


Found in line 386 at 2023-08-good-entry/contracts/GeVault.sol:

    if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);


Found in line 409 at 2023-08-good-entry/contracts/GeVault.sol:

    uint bal = t.balanceOf(address(this));


Found in line 412 at 2023-08-good-entry/contracts/GeVault.sol:

      lendingPool.deposit(address(t), bal, address(this), 0);


Found in line 423 at 2023-08-good-entry/contracts/GeVault.sol:

    liquidity = ERC20(aTokenAddress).balanceOf(address(this));


Found in line 138 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);


Found in line 139 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);


Found in line 153 at 2023-08-good-entry/contracts/TokenisableRange.sol:

         recipient: address(this),


Found in line 159 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    TOKEN0.token.safeTransfer( msg.sender,  TOKEN0.token.balanceOf(address(this)));


Found in line 160 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    TOKEN1.token.safeTransfer(msg.sender, TOKEN1.token.balanceOf(address(this)));


Found in line 171 at 2023-08-good-entry/contracts/TokenisableRange.sol:

        recipient: address(this),


Found in line 228 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    TOKEN0.token.transferFrom(msg.sender, address(this), n0);


Found in line 229 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    TOKEN1.token.transferFrom(msg.sender, address(this), n1);


Found in line 96 at 2023-08-good-entry/contracts/RangeManager.sol:

    ASSET_0.safeTransferFrom(msg.sender, address(this), amount0);


Found in line 98 at 2023-08-good-entry/contracts/RangeManager.sol:

    ASSET_1.safeTransferFrom(msg.sender, address(this), amount1);


Found in line 101 at 2023-08-good-entry/contracts/RangeManager.sol:

    ERC20(tr).safeTransfer(msg.sender, TokenisableRange(tr).balanceOf(address(this)));


Found in line 118 at 2023-08-good-entry/contracts/RangeManager.sol:

        trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));


Found in line 130 at 2023-08-good-entry/contracts/RangeManager.sol:

        uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));


Found in line 155 at 2023-08-good-entry/contracts/RangeManager.sol:

      LENDING_POOL.withdraw( address(ASSET_0), amount0, address(this) );


Found in line 160 at 2023-08-good-entry/contracts/RangeManager.sol:

      LENDING_POOL.withdraw( address(ASSET_1), amount1, address(this) );


Found in line 191 at 2023-08-good-entry/contracts/RangeManager.sol:

    uint256 asset0_amt = ASSET_0.balanceOf(address(this));


Found in line 192 at 2023-08-good-entry/contracts/RangeManager.sol:

    uint256 asset1_amt = ASSET_1.balanceOf(address(this));


Found in line 95 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        token.safeTransfer(msg.sender, token.balanceOf( address(this) ) );  // msg.sender has been Required to be owner


Found in line 115 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);


Found in line 127 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);


Found in line 132 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeTransfer(msg.sender, ogInAsset.balanceOf(address(this)));


Found in line 164 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);


Found in line 167 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountOut, amountInMax, 0));         


Found in line 182 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);


Found in line 186 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountIn, amountOutMin, 0));


Found in line 92 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    uint[2] memory amts = [ERC20(token0).balanceOf(address(this)), ERC20(token1).balanceOf(address(this))];


Found in line 103 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      uint debt = closeDebt(poolId, address(this), debtAsset, amount, collateral);


Found in line 104 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      uint amt0 = ERC20(token0).balanceOf(address(this));


Found in line 105 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      uint amt1 = ERC20(token1).balanceOf(address(this));


Found in line 176 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    LP.flashLoan( address(this), options, amounts, flashtype, msg.sender, params, 0);


Found in line 207 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    LP.flashLoan( address(this), options, amounts, flashtype, msg.sender, params, 0);


Found in line 275 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      if (user != address(this) ){


Found in line 310 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    if (user != address(this) ) LP.repay( debtAsset, debt, 2, user);


Found in line 312 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      uint amt0 = ERC20(token0).balanceOf(address(this));


Found in line 313 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      uint amt1 = ERC20(token1).balanceOf(address(this));


Found in line 369 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral");


Found in line 479 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

        address(this), 


Found in line 496 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( amountsIn[0] <= ERC20(path[0]).balanceOf(address(this)), "OPM: Insufficient Token Amount" );


Found in line 501 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      address(this),


Found in line 81 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);


Found in line 90 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    uint amt = ERC20(asset).balanceOf(address(this));


Found in line 115 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    ERC20(asset).safeTransfer(ROEROUTER.treasury(), ERC20(asset).balanceOf(address(this))); 


Found in line 127 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

      LP.withdraw(asset, amount, address(this));

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 12 

## [GAS] Functions guaranteed to revert when called by normal users can be marked payable
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 75 at 2023-08-good-entry/contracts/RangeManager.sol:

  function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {


Found in line 95 at 2023-08-good-entry/contracts/RangeManager.sol:

  function initRange(address tr, uint amount0, uint amount1) external onlyOwner {


Found in line 24 at 2023-08-good-entry/contracts/helper/FixedOracle.sol:

    function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {


Found in line 94 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

    function emergencyWithdraw(ERC20 token) onlyOwner external {   

------------------------------------------------------------------------ 

### Mitigation 

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.










# VULN 13 

## [GAS] Empty blocks should be removed or emit something
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 25 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

  constructor (address roerouter) PositionManager(roerouter) {}

------------------------------------------------------------------------ 

### Mitigation 

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}).










# VULN 14 

## [GAS] Use double require instead of using &&
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 120 at 2023-08-good-entry/contracts/GeVault.sol:

    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");



Found in line 141 at 2023-08-good-entry/contracts/GeVault.sol:

    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");



Found in line 170 at 2023-08-good-entry/contracts/GeVault.sol:

    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");



Found in line 209 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");



Found in line 271 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");



Found in line 108 at 2023-08-good-entry/contracts/RangeManager.sol:

    require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");



Found in line 29 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");



Found in line 101 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");



Found in line 23 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    require(clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");



Found in line 167 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");



Found in line 495 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );



Found in line 523 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require ( priceAssetA > 0 && priceAssetB > 0, "OPM: Invalid Oracle Price");



Found in line 536 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");


------------------------------------------------------------------------ 

### Mitigation 

Using double require instead of operator && can save more gas. When having a require statement with 2 or more expressions needed, place the expression that cost less gas first.










# VULN 15 

## [GAS] Use assembly to write address storage values
------------------------------------------------------------------------ 

### Proof of concept 

Found in lines 67 to 76 at 2023-08-good-entry/contracts/GeVault.sol:

  constructor(
    address _treasury, 
    address roeRouter, 
    address _uniswapPool, 
    uint poolId, 
    string memory name, 
    string memory symbol,
    address weth,
    bool _baseTokenIsToken0
  ) 

------------------------------------------------------------------------ 

### Mitigation 

Use assembly to write address storage values. Here are a few reasons: * Reduced opcode usage: When using assembly, you can directly manipulate storage values using lower-level instructions like sstore (storage store) instead of relying on higher-level Solidity storage assignments. These direct operations typically result in fewer opcode executions, reducing gas costs. * Avoiding unnecessary checks: Solidity storage assignments often involve additional checks and operations, such as enforcing security modifiers or triggering events. By using assembly, you can bypass these additional checks and perform the necessary storage operations directly, resulting in gas savings. * Optimized packing: Assembly provides greater flexibility in packing and unpacking data structures. By carefully arranging and manipulating the storage layout in assembly, you can achieve more efficient storage utilization and minimize wasted storage space. * Fine-grained control: Assembly allows for precise control over gas-consuming operations. You can optimize gas usage by selecting specific instructions and minimizing unnecessary operations or data copying.










# VULN 16 

## [GAS] require() or revert() statements that check input arguments should be at the top of the function (Also restructured some if’s)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 79 at 2023-08-good-entry/contracts/GeVault.sol:

    require(_treasury != address(0x0), "GEV: Invalid Treasury");


Found in line 80 at 2023-08-good-entry/contracts/GeVault.sol:

    require(_uniswapPool != address(0x0), "GEV: Invalid Pool");


Found in line 81 at 2023-08-good-entry/contracts/GeVault.sol:

    require(weth != address(0x0), "GEV: Invalid WETH");


Found in line 125 at 2023-08-good-entry/contracts/GeVault.sol:

        require( t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap");


Found in line 127 at 2023-08-good-entry/contracts/GeVault.sol:

        require( t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap");


Found in line 146 at 2023-08-good-entry/contracts/GeVault.sol:

        require( t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");


Found in line 148 at 2023-08-good-entry/contracts/GeVault.sol:

        require( t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");


Found in line 256 at 2023-08-good-entry/contracts/GeVault.sol:

      require(token == address(WETH), "GEV: Invalid Weth");


Found in line 269 at 2023-08-good-entry/contracts/GeVault.sol:

    require(tvlCap > valueX8 + getTVL(), "GEV: Max Cap Reached");


Found in line 281 at 2023-08-good-entry/contracts/GeVault.sol:

    require(liquidity > 0, "GEV: No Liquidity Added");


Found in line 34 at 2023-08-good-entry/contracts/RoeRouter.sol:

    require(treasury_ != address(0x0), "Invalid address");


Found in line 75 at 2023-08-good-entry/contracts/RoeRouter.sol:

    require(token0 < token1, "Invalid Order");


Found in line 209 at 2023-08-good-entry/contracts/TokenisableRange.sol:

      require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");


Found in line 49 at 2023-08-good-entry/contracts/RangeManager.sol:

    require( address(lendingPool) != address(0x0), "Invalid address" );


Found in line 66 at 2023-08-good-entry/contracts/RangeManager.sol:

      revert("Range overlap");


Found in line 206 at 2023-08-good-entry/contracts/RangeManager.sol:

    require(hf > 1.01e18, "Health factor is too low");


Found in line 29 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");


Found in line 63 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    require(timeStamp > 0, "Round not complete");


Found in line 101 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

    require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");


Found in line 23 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    require(clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");


Found in line 26 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    require(CL_TOKENA.decimals() + CL_TOKENB.decimals() >= 16, "Decimals error");


Found in line 44 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    require(timeStamp > 0, "Round not complete");


Found in line 91 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

       require(acceptPayable, "CannotReceiveETH");


Found in line 69 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");


Found in line 91 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");


Found in line 137 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

      require(sourceSwap == token0 || sourceSwap == token1, "OPM: Invalid Swap Token");


Found in line 167 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");


Found in line 198 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(options.length == amounts.length, "ARRAY_LEN_MISMATCH");


Found in line 235 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(debt > 0, "OPM: No Debt");


Found in line 261 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(collateralAsset == token0 || collateralAsset == token1, "OPM: Invalid Collateral Asset");


Found in line 344 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( 


Found in line 369 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral");


Found in line 393 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );


Found in line 420 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );


Found in line 525 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    require( amountB > 0, "OPM: Target Amount Too Low");


Found in line 30 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    require(roerouter != address(0x0), "Invalid address");


Found in line 145 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    require( valueA <= valueB * 101 / 100, "PM: LP Oracle Error");


Found in line 146 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    require( valueB <= valueA * 101 / 100, "PM: LP Oracle Error");

------------------------------------------------------------------------ 

### Mitigation 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting alot of gas in a function that may ultimately revert in the unhappy case.
