No | Issue |Instances|
|-|:-|:-:|:-:|
|G-01|Use hardcode address instead address(this)|40
|G-02|Use function instead of modifiers |1
|G-03|Use assembly to check for address(0)|1
|G-04|Using calldata instead of memory for read-only arguments in external functions saves gas|8
|G-05|Use assembly to perform efficient back-to-back calls|1
|G-06|Use nested if and, avoid multiple check combinations|8
|G-07|Do not calculate constant|6
|G-08|Unnecessary look up in if condition|4
|G-09|Bytes constants are more efficient than String constants|7
|G-10|Use do while loops instead of for loops|9

### [G-01] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

```solidity

file: PositionManager/OptionsPositionManager.sol

92   uint[2] memory amts = [ERC20(token0).balanceOf(address(this)), ERC20(token1).balanceOf(address(this))];

103  uint debt = closeDebt(poolId, address(this), debtAsset, amount, collateral);
      uint amt0 = ERC20(token0).balanceOf(address(this));
105      uint amt1 = ERC20(token1).balanceOf(address(this));

176   LP.flashLoan( address(this), options, amounts, flashtype, msg.sender, params, 0);

207  LP.flashLoan( address(this), options, amounts, flashtype, msg.sender, params, 0);

275   if (user != address(this) )

310  if (user != address(this) ) LP.repay( debtAsset, debt, 2, user);
    {
      uint amt0 = ERC20(token0).balanceOf(address(this));
313      uint amt1 = ERC20(token1).balanceOf(address(this));

369    require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral");

479   address(this), 

496     require( amountsIn[0] <= ERC20(path[0]).balanceOf(address(this)), "OPM: Insufficient Token Amount" );
    amountsIn = ammRouter.swapTokensForExactTokens(
      recvAmount,
      maxAmount,
      path,
501      address(this),

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L92
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L103-L105
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L176
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L207
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L275
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L310-L313
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L369
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L479
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L496-L501

```solidity

file: GeVault.sol

262   ERC20(token).safeTransferFrom(msg.sender, address(this), amount);

302   uint bal = ERC20(aTick).balanceOf(address(this));

324  uint aBal = ERC20(aTokenAddress).balanceOf(address(this));

330   lendingPool.withdraw(address(tr), aBal, address(this));

339  uint availToken0 = token0.balanceOf(address(this));
340    uint availToken1 = token1.balanceOf(address(this));

386  if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

409   uint bal = t.balanceOf(address(this));
    if (bal > 0){
      checkSetApprove(address(t), address(lendingPool), bal);
412      lendingPool.deposit(address(t), bal, address(this), 0);

423   liquidity = ERC20(aTokenAddress).balanceOf(address(this));
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L262
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L302
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L324
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L330
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L339-L340
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L386
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L409-L412
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L423

```solidity

file: TokenisableRange.sol

138 TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);
139    TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);

153     recipient: address(this),

159  TOKEN0.token.safeTransfer( msg.sender,  TOKEN0.token.balanceOf(address(this)));
160    TOKEN1.token.safeTransfer(msg.sender, TOKEN1.token.balanceOf(address(this)));

171   recipient: address(this),

228  TOKEN0.token.transferFrom(msg.sender, address(this), n0);
229    TOKEN1.token.transferFrom(msg.sender, address(this), n1);
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L138
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L153
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L159-L160
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L171
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L228-L229

```solidity

file: helper/V3Proxy.sol

95   token.safeTransfer(msg.sender, token.balanceOf( address(this) ) ); 

115  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);

127    ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);

132  ogInAsset.safeTransfer(msg.sender, ogInAsset.balanceOf(address(this)));

164 ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax);

167    amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, address(this),       deadline, amountOut, amountInMax, 0));   

182  ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn);

186    amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountIn, amountOutMin, 0));
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L95
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L115
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L127
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L132
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L164
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L167
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L182
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L186

```solidity

file: RangeManager.sol

95   function initRange(address tr, uint amount0, uint amount1) external onlyOwner {
    ASSET_0.safeTransferFrom(msg.sender, address(this), amount0);
    ASSET_0.safeIncreaseAllowance(tr, amount0);
    ASSET_1.safeTransferFrom(msg.sender, address(this), amount1);
    ASSET_1.safeIncreaseAllowance(tr, amount1);
    TokenisableRange(tr).init(amount0, amount1);
    ERC20(tr).safeTransfer(msg.sender, TokenisableRange(tr).balanceOf(address(this)));
102  }

118    trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));

130    uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));

155   LENDING_POOL.withdraw( address(ASSET_0), amount0, address(this) 

160  LENDING_POOL.withdraw( address(ASSET_1), amount1, address(this) );

191  uint256 asset0_amt = ASSET_0.balanceOf(address(this));
192   uint256 asset1_amt = ASSET_1.balanceOf(address(this));
    
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L95-L102
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L118
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L130
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L155
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L160
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L191-L192

```solidity

file: PositionManager/PositionManager.sol

81  if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

90  uint amt = ERC20(asset).balanceOf(address(this));

115   ERC20(asset).safeTransfer(ROEROUTER.treasury(), ERC20(asset).balanceOf(address(this))); 

127  LP.withdraw(asset, amount, address(this));
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L81
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L90
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L115
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L127

### [G-02] Use function instead of modifiers 

```solidity

file: helper/FixedOracle.sol

10    modifier onlyOwner() {
        require(msg.sender == owner, "Only the owner can call this function.");
        _;
    }


```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L10

### [G-03] Use assembly to check for address(0)
Saves 6 gas per instance

```solidity

file: PositionManager/OptionsPositionManager.sol

136  if (sourceSwap != address(0) )

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L136

## [G‑04] Using calldata instead of memory for read-only arguments in external functions saves gas
When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas

```solidity

file: PositionManager/OptionsPositionManager.sol

8  function getAmountsOut(uint256 amountIn, address[] calldata path) external returns (uint256[] memory amounts);
9  function getAmountsIn(uint256 amountOut, address[] calldata path) external returns (uint256[] memory amounts);

61   function executeBuyOptions(
    uint poolId,
    address[] calldata assets,
    uint256[] calldata amounts,
    address user,
    address[] memory sourceSwap
67  )

159  function buyOptions(
    uint poolId, 
    address[] memory options, 
    uint[] memory amounts, 
    address[] memory sourceSwap
164  )

189   function liquidate (
    uint poolId, 
    address user,
    address[] memory options, 
    uint[] memory amounts,
    address collateralAsset
195  )

470  function swapExactTokensForTokens(IUniswapV2Router01 ammRouter, IPriceOracle oracle, uint amount, address[] memory path) 

491   function swapTokensForExactTokens(IUniswapV2Router01 ammRouter, uint recvAmount, uint maxAmount, address[] memory path) internal 
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L8-L9
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L61-L67
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L159-164
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L189-L95
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L470
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L491

```solidity

file: TokenisableRange.sol

85 function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external 

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L85

```solidity

file: RangeManager.sol

75  function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner 

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75

### [G-05] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

```solidity

file: PositionManager/OptionsPositionManager.sol

8 function getAmountsOut(uint256 amountIn, address[] calldata path) external returns (uint256[] memory amounts);
9  function getAmountsIn(uint256 amountOut, address[] calldata path) external returns (uint256[] memory amounts);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L8-L9

### [G-06] Use nested if and, avoid multiple check combinations

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity

file: PositionManager/OptionsPositionManager.sol

234  if ( repayAmount > 0 && repayAmount < debt ) debt = repayAmount;

473 if (amount > 0 && AmountsRouter(address(ammRouter)).getAmountsOut(amount, path)[1] > 0)
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L234
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L473

```solidity

file: GeVault.sol

377     if (oraclePrice < priceX8 * 101 / 100 && oraclePrice > priceX8 * 99 / 100) matches = true;
  }

434    if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )  
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L377
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L434

```solidity

file: TokenisableRange.sol

177   if ((newFee0 == 0) && (newFee1 == 0)) return;  

190  if ((fee0 * 100 > bal0 ) && (fee1 * 100 > bal1)) 

237     if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) )

268  if ( newFee0 == 0 && newFee1 == 0 )
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L177
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L190
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L237
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L268

## [G-07] Do not calculate constant

When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.

```solidity

file: PositionManager/OptionsPositionManager.sol

346 || (tokensValue > debtValue * 98 / 100 && tokensValue < debtValue * 102 / 100),

365 uint feeValueE8 = token0Amount * oracle.getAssetPrice(token0) / 10**ERC20(token0).decimals()
                    + token1Amount * oracle.getAssetPrice(token1) / 10**ERC20(token1).decimals() ;
367    feeAmount = feeValueE8 * 10**ERC20(collateralAsset).decimals() / 100 / oracle.getAssetPrice(collateralAsset);

524  amountB = amountA * priceAssetA * 10**ERC20(assetB).decimals() / 10**ERC20(assetA).decimals() / priceAssetB;

546   uint scale0 = 10**(20 - ERC20(token0).decimals()) * oracle.getAssetPrice(token0) / 1e8;
547    uint scale1 = 10**(20 - ERC20(token1).decimals()) * oracle.getAssetPrice(token1) / 1e8;
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L346
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L365-L367
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L524
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L546

```solidity

file: TokenisableRange.sol

99     int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );
    int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );
    
    if (isTicker) { 
      feeTier   = 5;
      int24 midleTick;
      midleTick = (_upperTick + _lowerTick) / 2;
      _upperTick = (midleTick + int24(feeTier)) - (midleTick + int24(feeTier)) % int24(feeTier * 2);
      _lowerTick = _upperTick - int24(feeTier) - int24(feeTier);
      _name     = string(abi.encodePacked("Ticker ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));
     _symbol    = string(abi.encodePacked("T-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));
    } else {
      feeTier   = 5;
      _lowerTick = (_lowerTick + int24(feeTier)) - (_lowerTick + int24(feeTier)) % int24(feeTier * 2);
      _upperTick = (_upperTick + int24(feeTier)) - (_upperTick + int24(feeTier)) % int24(feeTier * 2);
      _name     = string(abi.encodePacked("Ranger ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));
      _symbol   = string(abi.encodePacked("R-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));
    }
    lowerTick = _lowerTick;
    upperTick = _upperTick;
    emit InitTR(address(asset0), address(asset1), startX10, endX10);
116  }
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L99-L116

```solidity

file: 

89     if (decimalsB >= decimalsA) {
      norm_b = sqrt( a * b * priceA * 10**(decimalsB-decimalsA) / priceB );
    } else {
      norm_b = sqrt( a * b * priceA / 10**(decimalsA-decimalsB) / priceB );
    }
    uint norm_a = a * b / norm_b;

    /*
      The normalised positions (18 decimals) are multiplied with the chainlink value (8 decimals), giving val.
      val is divided by LP_TOKEN.totalSupply(), which has 18 decimals, and casted to an int
      The return value represents the value * 10**8 of a single LP token 
    */
    require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");
102    uint val = norm_a * priceA * 10**(18-decimalsA) + norm_b * 10**(18-decimalsB) * priceB;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L89-L102

### [G-08]  Unnecessary look up in if condition
If the || condition isn’t required, the second condition will have been looked up unnecessarily.

```solidity

file: PositionManager/OptionsPositionManager.sol

137    require(sourceSwap == token0 || sourceSwap == token1, "OPM: Invalid Swap Token");


```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L137

```solidity

file: contracts/GeVault.sol

274 if (tSupply == 0 || vaultValueX8 == 0)

434     if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L274
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L434

```solidity

file: contracts/TokenisableRange.sol

237  if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) )
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L237

### [G-09] Bytes constants are more efficient than String constants
If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

```solidity

file: contracts/GeVault.sol

72  string memory name, 
73    string memory symbol,
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L72-L73

```solidity

file: TokenisableRange.sol

45 string _name;
46  string _symbol;

85  function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external 

96  string memory quoteSymbol = asset0.symbol();
97   string memory baseSymbol  = asset1.symbol();

107    _name     = string(abi.encodePacked("Ticker ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));
108    _symbol    = string(abi.encodePacked("T-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));

114   _name     = string(abi.encodePacked("Ranger ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));
115      _symbol   = string(abi.encodePacked("R-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));

125 function name()     public view virtual override returns (string memory) { return _name; }
  /// @notice Get the symbol of this contract token
127 function symbol()   public view virtual override returns (string memory) { return _symbol; }
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L45-L46
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L85
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L96-L97
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L107-L108
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L114-L115
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L125-L127

### [G-10] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity


file: PositionManager/OptionsPositionManager.sol


71   for ( uint8 k = 0; k<assets.length; k++)

93   for ( uint8 k =0; k<assets.length; k++)

172  for (uint8 i = 0; i< options.length; )

203   for (uint8 i = 0; i< options.length; )
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L71
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L93
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L172
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L203

```solidity

file: contracts/GeVault.sol

154   for (uint k = 0; k < ticks.length - 2; k++) 

299  for (uint k = 0; k < ticks.length; k++)

314  for (uint k = 0; k < ticks.length; k++)

393  for(uint k=0; k<ticks.length; k++)

431   for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++)
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L154
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L299
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L314
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L393
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L431