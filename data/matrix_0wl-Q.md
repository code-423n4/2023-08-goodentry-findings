## Non Critical Issues

|       | Issue                                                                   |
| ----- | :---------------------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                    |
| NC-2  | ADD TO BLACKLIST FUNCTION                                               |
| NC-3  | Be explicit declaring types                                             |
| NC-4  | COMMENTED CODE                                                          |
| NC-5  | INCONSISTENT SOLIDITY VERSIONS                                          |
| NC-6  | FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES                 |
| NC-7  | DON’T WORRY ABOUT KECCAK256 COLLISIONS                                  |
| NC-8  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION           |
| NC-9  | NO SAME VALUE INPUT CONTROL                                             |
| NC-10 | Omissions in events + Add parameter to event-emit                       |
| NC-11 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS |
| NC-12 | LINES ARE TOO LONG                                                      |
| NC-13 | Constants should be defined rather than using magic numbers             |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

101:   function setEnabled(bool _isEnabled) public onlyOwner {

108:   function setTreasury(address newTreasury) public onlyOwner {

183:   function setBaseFee(uint newBaseFeeX4) public onlyOwner {

191:   function setTvlCap(uint newTvlCap) public onlyOwner {

```

```solidity
File: /contracts/RoeRouter.sol

83:   function setTreasury(address newTreasury) public onlyOwner {

```

```solidity
File: /contracts/helper/FixedOracle.sol

24:     function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {

```

### [NC-2] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/RangeManager.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/RoeRouter.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/TokenisableRange.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/helper/FixedOracle.sol

1: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/helper/LPOracle.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/helper/V3Proxy.sol

3: pragma solidity 0.8.19;

```

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

### [NC-3] Be explicit declaring types

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

30:   event Deposit(address indexed sender, address indexed token, uint amount, uint liquidity);

30:   event Deposit(address indexed sender, address indexed token, uint amount, uint liquidity);

31:   event Withdraw(address indexed sender, address indexed token, uint amount, uint liquidity);

31:   event Withdraw(address indexed sender, address indexed token, uint amount, uint liquidity);

34:   event ModifyTick(address indexed ticker, uint index);

35:   event Rebalance(uint tickIndex);

38:   event SetFee(uint baseFeeX4);

39:   event SetTvlCap(uint tvlCap);

46:   uint public tickIndex;

52:   uint public baseFeeX4 = 20;

54:   uint public tvlCap = 1e12;

62:   uint public constant nearbyRanges = 2;

71:     uint poolId,

154:         for (uint k = 0; k < ticks.length - 2; k++)

167:   function modifyTick(address tr, uint index) public onlyOwner {

177:   function getTickLength() public view returns(uint len){

183:   function setBaseFee(uint newBaseFeeX4) public onlyOwner {

191:   function setTvlCap(uint newTvlCap) public onlyOwner {

214:   function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {

214:   function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {

220:     uint vaultValueX8 = getTVL();

221:     uint valueX8 = vaultValueX8 * liquidity / totalSupply();

223:     uint fee = amount * getAdjustedBaseFee(token == address(token1)) / 1e4;

228:     uint bal = amount - fee;

247:   function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity)

247:   function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity)

266:     uint fee = amount * getAdjustedBaseFee(token == address(token0)) / 1e4;

268:     uint valueX8 = oracle.getAssetPrice(token) * (amount - fee) / 10**ERC20(token).decimals();

271:     uint vaultValueX8 = getTVL();

272:     uint tSupply = totalSupply();

290:     uint supply = totalSupply();

292:     uint vaultValue = getTVL();

298:   function getReserves() public view returns (uint amount0, uint amount1){

298:   function getReserves() public view returns (uint amount0, uint amount1){

299:     for (uint k = 0; k < ticks.length; k++){

302:       uint bal = ERC20(aTick).balanceOf(address(this));

303:       (uint amt0, uint amt1) = t.getTokenAmounts(bal);

303:       (uint amt0, uint amt1) = t.getTokenAmounts(bal);

314:     for (uint k = 0; k < ticks.length; k++){

321:   function removeFromTick(uint index) internal {

324:     uint aBal = ERC20(aTokenAddress).balanceOf(address(this));

325:     uint sBal = tr.balanceOf(aTokenAddress);

338:     uint newTickIndex = getActiveTickIndex();

339:     uint availToken0 = token0.balanceOf(address(this));

340:     uint availToken1 = token1.balanceOf(address(this));

343:     (uint amount0ft, uint amount1ft) = ticks[newTickIndex].getTokenAmountsExcludingFees(1e18);

343:     (uint amount0ft, uint amount1ft) = ticks[newTickIndex].getTokenAmountsExcludingFees(1e18);

344:     uint tick0Index = newTickIndex;

345:     uint tick1Index = newTickIndex + 2;

370:     uint decimals0 = token0.decimals();

371:     uint decimals1 = token1.decimals();

372:     uint priceX8 = 10**decimals0;

374:     priceX8 = priceX8 * uint(sqrtPriceX96 / 2 ** 12) ** 2 * 1e8 / 2**168;

376:     uint oraclePrice = 1e8 * oracle.getAssetPrice(address(token0)) / oracle.getAssetPrice(address(token1));

385:   function checkSetApprove(address token, address spender, uint amount) private {

392:   function getTVL() public view returns (uint valueX8){

393:     for(uint k=0; k<ticks.length; k++){

395:       uint bal = getTickBalance(k);

404:   function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity){

404:   function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity){

404:   function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity){

409:     uint bal = t.balanceOf(address(this));

420:   function getTickBalance(uint index) public view returns (uint liquidity) {

420:   function getTickBalance(uint index) public view returns (uint liquidity) {

428:   function getActiveTickIndex() public view returns (uint activeTickIndex) {

432:         (uint amt0, uint amt1) = ticks[activeTickIndex+1].getTokenAmountsExcludingFees(1e18);

432:         (uint amt0, uint amt1) = ticks[activeTickIndex+1].getTokenAmountsExcludingFees(1e18);

433:         (uint amt0n, uint amt1n) = ticks[activeTickIndex+2].getTokenAmountsExcludingFees(1e18);

433:         (uint amt0n, uint amt1n) = ticks[activeTickIndex+2].getTokenAmountsExcludingFees(1e18);

445:   function getAdjustedBaseFee(bool increaseToken0) public view returns (uint adjustedBaseFeeX4) {

446:     (uint res0, uint res1) = getReserves();

446:     (uint res0, uint res1) = getReserves();

447:     uint value0 = res0 * oracle.getAssetPrice(address(token0)) / 10**token0.decimals();

448:     uint value1 = res1 * oracle.getAssetPrice(address(token1)) / 10**token1.decimals();

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

16:   event BuyOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

16:   event BuyOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

16:   event BuyOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

17:   event SellOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

17:   event SellOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

17:   event SellOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

18:   event ClosePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

18:   event ClosePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

18:   event ClosePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

19:   event LiquidatePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

19:   event LiquidatePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

19:   event LiquidatePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

20:   event ReducedPosition(address indexed user, address indexed asset, uint amount);

21:   event Swap(address indexed user, address sourceAsset, uint sourceAmount, address targetAsset, uint targetAmount);

21:   event Swap(address indexed user, address sourceAsset, uint sourceAmount, address targetAsset, uint targetAmount);

48:       (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));

48:       (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));

53:       (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));

53:       (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));

62:     uint poolId,

73:       uint amount = amounts[k];

84:     uint poolId,

92:     uint[2] memory amts = [ERC20(token0).balanceOf(address(this)), ERC20(token1).balanceOf(address(this))];

97:       uint amount = amounts[k];

103:       uint debt = closeDebt(poolId, address(this), debtAsset, amount, collateral);

104:       uint amt0 = ERC20(token0).balanceOf(address(this));

105:       uint amt1 = ERC20(token1).balanceOf(address(this));

124:     uint poolId,

141:       uint amount = sourceSwap == token0 ? amount0 : amount1;

143:       uint received = swapExactTokensForTokens(router, oracle, amount, path);

160:     uint poolId,

162:     uint[] memory amounts,

171:     uint[] memory flashtype = new uint[](options.length);

171:     uint[] memory flashtype = new uint[](options.length);

190:     uint poolId,

193:     uint[] memory amounts,

202:     uint[] memory flashtype = new uint[](options.length);

202:     uint[] memory flashtype = new uint[](options.length);

224:     uint poolId,

227:     uint repayAmount,

233:     uint debt = ERC20(LP.getReserveData(debtAsset).variableDebtTokenAddress).balanceOf(user);

251:     uint poolId,

254:     uint repayAmount,

257:     internal returns (uint debt)

262:     uint amtA;

263:     uint amtB;

270:       (uint token0Amount, uint token1Amount) = TokenisableRange(debtAsset).getTokenAmounts(debt);

270:       (uint token0Amount, uint token1Amount) = TokenisableRange(debtAsset).getTokenAmounts(debt);

282:           uint feeAmount = calculateAndSendFee(poolId, token0Amount, token1Amount, collateralAsset);

312:       uint amt0 = ERC20(token0).balanceOf(address(this));

313:       uint amt1 = ERC20(token1).balanceOf(address(this));

336:   function checkExpectedBalances(address debtAsset, uint debtAmount, uint token0Amount, uint token1Amount) internal view

336:   function checkExpectedBalances(address debtAsset, uint debtAmount, uint token0Amount, uint token1Amount) internal view

336:   function checkExpectedBalances(address debtAsset, uint debtAmount, uint token0Amount, uint token1Amount) internal view

341:     uint debtValue = TokenisableRange(debtAsset).latestAnswer() * debtAmount / 1e18;

342:     uint tokensValue = token0Amount * oracle.getAssetPrice(address(token0)) / 10**decimals0 + token1Amount * oracle.getAssetPrice(address(token1)) / 10**decimals1;

358:     uint poolId,

359:     uint token0Amount,

360:     uint token1Amount,

362:   ) internal returns (uint feeAmount) {

365:     uint feeValueE8 = token0Amount * oracle.getAssetPrice(token0) / 10**ERC20(token0).decimals()

385:     uint poolId,

387:     uint amount0,

388:     uint amount1

399:     uint deposited = TokenisableRange(optionAddress).deposit(amount0, amount1);

413:     uint poolId,

415:     uint amount

424:     (uint amount0, uint amount1) = TokenisableRange(optionAddress).withdraw(amount, 0, 0);

424:     (uint amount0, uint amount1) = TokenisableRange(optionAddress).withdraw(amount, 0, 0);

439:   function swapTokens(uint poolId, address sourceAsset, uint amount) public returns (uint received) {

439:   function swapTokens(uint poolId, address sourceAsset, uint amount) public returns (uint received) {

439:   function swapTokens(uint poolId, address sourceAsset, uint amount) public returns (uint received) {

470:   function swapExactTokensForTokens(IUniswapV2Router01 ammRouter, IPriceOracle oracle, uint amount, address[] memory path)

475:       uint[] memory amounts = ammRouter.swapExactTokensForTokens(

491:   function swapTokensForExactTokens(IUniswapV2Router01 ammRouter, uint recvAmount, uint maxAmount, address[] memory path) internal {

491:   function swapTokensForExactTokens(IUniswapV2Router01 ammRouter, uint recvAmount, uint maxAmount, address[] memory path) internal {

493:     uint [] memory amountsIn = AmountsRouter(address(ammRouter)).getAmountsIn(recvAmount, path);

513:   function getTargetAmountFromOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB)

514:     internal view returns (uint amountB)

517:       uint valueA = amountA * oracle.getAssetPrice(assetA) / 10**ERC20(assetA).decimals();

518:       uint valueB = amountB * oracle.getAssetPrice(assetB) / 10**ERC20(assetB).decimals();

521:     uint priceAssetA = oracle.getAssetPrice(assetA);

522:     uint priceAssetB = oracle.getAssetPrice(assetB);

544:   function addDust(address debtAsset, address token0, address token1) internal returns (uint amount){

546:     uint scale0 = 10**(20 - ERC20(token0).decimals()) * oracle.getAssetPrice(token0) / 1e8;

547:     uint scale1 = 10**(20 - ERC20(token1).decimals()) * oracle.getAssetPrice(token1) / 1e8;

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

63:   function getPoolAddresses(uint poolId)

80:   function checkSetAllowance(address token, address spender, uint amount) internal {

90:     uint amt = ERC20(asset).balanceOf(address(this));

95:       uint debt = ERC20(LP.getReserveData(asset).variableDebtTokenAddress).balanceOf(user);

124:   function PMWithdraw(ILendingPool LP, address user, address asset, uint amount) internal {

138:   function validateValuesAgainstOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB, uint amountB) internal view {

138:   function validateValuesAgainstOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB, uint amountB) internal view {

139:     uint decimalsA = ERC20(assetA).decimals();

140:     uint decimalsB = ERC20(assetB).decimals();

141:     uint valueA = amountA * oracle.getAssetPrice(assetA);

142:     uint valueB = amountB * oracle.getAssetPrice(assetB);

```

```solidity
File: /contracts/RangeManager.sol

28:   event Withdraw(address user, address asset, uint amount);

29:   event Deposit(address user, address asset, uint amount);

30:   event AddRange(uint128 startX10, uint128 endX10, uint step);

62:     for (uint i = 0; i < len; i++) {

95:   function initRange(address tr, uint amount0, uint amount1) external onlyOwner {

95:   function initRange(address tr, uint amount0, uint amount1) external onlyOwner {

```

```solidity
File: /contracts/RoeRouter.sol

12:   event AddPool(uint poolId, address lendingPoolAddressProvider);

13:   event DeprecatePool(uint poolId);

40:   function getPoolsLength() public view returns (uint poolLength) {

48:   function deprecatePool(uint poolId) public onlyOwner {

66:     returns (uint poolId)

```

```solidity
File: /contracts/TokenisableRange.sol

22:   event Deposit(address sender, uint trAmount);

23:   event Withdraw(address sender, uint trAmount);

24:   event ClaimFees(uint fee0, uint fee1);

24:   event ClaimFees(uint fee0, uint fee1);

55:   uint public treasuryFee_deprecated = 20;

61:   uint constant public treasuryFee = 20;

66:   function sqrt(uint x) internal pure returns (uint y) {

66:   function sqrt(uint x) internal pure returns (uint y) {

67:       uint z = (x + 1) / 2;

134:   function init(uint n0, uint n1) external {

134:   function init(uint n0, uint n1) external {

178:     uint tf0 = newFee0 * treasuryFee / 100;

179:     uint tf1 = newFee1 * treasuryFee / 100;

204:       uint token0Price = ORACLE.getAssetPrice(address(TOKEN0.token));

205:       uint token1Price = ORACLE.getAssetPrice(address(TOKEN1.token));

206:       uint addedValue = added0 * token0Price / 10**TOKEN0.decimals + added1 * token1Price / 10**TOKEN1.decimals;

207:       uint totalValue =   bal0 * token0Price / 10**TOKEN0.decimals +   bal1 * token1Price / 10**TOKEN1.decimals;

208:       uint liquidityValue = totalValue * newLiquidity / liquidity;

231:     uint newFee0;

232:     uint newFee1;

294:     uint removedLiquidity = uint(liquidity) * lp / totalSupply();

294:     uint removedLiquidity = uint(liquidity) * lp / totalSupply();

333:   function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {

333:   function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {

343:   function returnExpectedBalance(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 amt0, uint256 amt1) {

343:   function returnExpectedBalance(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 amt0, uint256 amt1) {

352:   function getValuePerLPAtPrice(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 priceX1e8) {

352:   function getValuePerLPAtPrice(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 priceX1e8) {

355:     uint totalValue = TOKEN0_PRICE * amt0 / (10 ** TOKEN0.decimals) + amt1 * TOKEN1_PRICE / (10 ** TOKEN1.decimals);

368:   function getTokenAmountsExcludingFees(uint amount) public view returns (uint token0Amount, uint token1Amount){

368:   function getTokenAmountsExcludingFees(uint amount) public view returns (uint token0Amount, uint token1Amount){

368:   function getTokenAmountsExcludingFees(uint amount) public view returns (uint token0Amount, uint token1Amount){

371:     (token0Amount, token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  uint128 ( uint(liquidity) * amount / totalSupply() ) );

377:   function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){

377:   function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){

377:   function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){

```

```solidity
File: /contracts/helper/LPOracle.sol

7:   function totalSupply() external view returns (uint);

44:   function sqrt(uint x) internal pure returns (uint y) {

44:   function sqrt(uint x) internal pure returns (uint y) {

45:     uint z = (x + 1) / 2;

61:       uint timeStamp,

70:     (uint a, uint b,) = LP_TOKEN.getReserves();

70:     (uint a, uint b,) = LP_TOKEN.getReserves();

72:     uint priceA = uint(getAnswer(CL_TOKENA));

72:     uint priceA = uint(getAnswer(CL_TOKENA));

73:     uint priceB = uint(getAnswer(CL_TOKENB));

73:     uint priceB = uint(getAnswer(CL_TOKENB));

88:     uint norm_b;

94:     uint norm_a = a * b / norm_b;

102:     uint val = norm_a * priceA * 10**(18-decimalsA) + norm_b * 10**(18-decimalsB) * priceB;

```

```solidity
File: /contracts/helper/V3Proxy.sol

57:     function withdraw(uint) external;

98:     function getAmountsOut(uint amountIn, address[] calldata path) external returns (uint[] memory amounts) {

98:     function getAmountsOut(uint amountIn, address[] calldata path) external returns (uint[] memory amounts) {

100:         amounts = new uint[](2);

105:     function getAmountsIn(uint amountOut, address[] calldata path) external returns (uint[] memory amounts) {

105:     function getAmountsIn(uint amountOut, address[] calldata path) external returns (uint[] memory amounts) {

107:         amounts = new uint[](2);

112:     function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

112:     function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

112:     function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

112:     function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

117:         amounts = new uint[](2);

124:     function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

124:     function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

124:     function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

124:     function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

129:         amounts = new uint[](2);

137:     function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

137:     function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

137:     function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

140:         amounts = new uint[](2);

147:     function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

147:     function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

147:     function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

150:         amounts = new uint[](2);

160:     function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

160:     function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

160:     function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

160:     function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

166:         amounts = new uint[](2);

178:     function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

178:     function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

178:     function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

178:     function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

184:         amounts = new uint[](2);

```

### [NC-4] COMMENTED CODE

#### **Proof Of Concept**

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

199:     bytes memory params = abi.encode(1, poolId, user, collateralAsset); // mode = 1 -> liquidation

204:       flashtype[i] = 0; // dont open debt for liquidations, need to repay

269:     { //localize vars

477:         getTargetAmountFromOracle(oracle, path[0], amount, path[1]) * 99 / 100, // allow 1% slippage

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

21:   ILendingPoolAddressesProvider public ADDRESSES_PROVIDER; // IFlashLoanReceiver  requirement

22:   ILendingPool public LENDING_POOL; // IFlashLoanReceiver  requirement

```

```solidity
File: /contracts/helper/V3Proxy.sol

95:         token.safeTransfer(msg.sender, token.balanceOf( address(this) ) );  // msg.sender has been Required to be owner

```

### [NC-5] INCONSISTENT SOLIDITY VERSIONS

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/RangeManager.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/RoeRouter.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/TokenisableRange.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/helper/FixedOracle.sol

1: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/helper/LPOracle.sol

2: pragma solidity 0.8.19;

```

```solidity
File: /contracts/helper/V3Proxy.sol

3: pragma solidity 0.8.19;

```

### [NC-6] FOR MODERN AND MORE READABLE CODE; UPDATE IMPORT USAGES

#### Description:

Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.

This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

4: import "./openzeppelin-solidity/contracts/access/Ownable.sol";

5: import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";

6: import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";

7: import "./openzeppelin-solidity/contracts/security/ReentrancyGuard.sol";

8: import "../interfaces/IAaveLendingPoolV2.sol";

9: import "../interfaces/IUniswapV3Pool.sol";

10: import "../interfaces/IWETH.sol";

11: import "./RangeManager.sol";

12: import "./RoeRouter.sol";

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

4: import "./PositionManager.sol";

5: import "../TokenisableRange.sol";

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

3: import "../openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";

4: import "../openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";

10: import "../../interfaces/IUniswapV2Router01.sol";

11: import "../../interfaces/IUniswapV2Pair.sol";

12: import "../../interfaces/IUniswapV2Factory.sol";

14: import "../RoeRouter.sol";

```

```solidity
File: /contracts/RangeManager.sol

4: import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";

5: import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";

6: import "./openzeppelin-solidity/contracts/utils/cryptography/ECDSA.sol";

7: import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";

8: import "./openzeppelin-solidity/contracts/security/ReentrancyGuard.sol";

9: import "../interfaces/AggregatorV3Interface.sol";

10: import "../interfaces/ILendingPoolAddressesProvider.sol";

11: import "../interfaces/IAaveLendingPoolV2.sol";

12: import "../interfaces/IAaveOracle.sol";

13: import "../interfaces/IUniswapV2Pair.sol";

14: import "../interfaces/IUniswapV2Factory.sol";

15: import "../interfaces/IUniswapV2Router01.sol";

16: import "../interfaces/ISwapRouter.sol";

17: import "../interfaces/INonfungiblePositionManager.sol";

18: import "./TokenisableRange.sol";

19: import "./openzeppelin-solidity/contracts/proxy/beacon/BeaconProxy.sol";

20: import "./openzeppelin-solidity/contracts/access/Ownable.sol";

```

```solidity
File: /contracts/RoeRouter.sol

4: import "./openzeppelin-solidity/contracts/access/Ownable.sol";

```

```solidity
File: /contracts/TokenisableRange.sol

4: import "../interfaces/INonfungiblePositionManager.sol";

5: import "../interfaces/IUniswapV3Factory.sol";

6: import "../interfaces/IUniswapV3Pool.sol";

7: import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";

8: import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";

9: import "./openzeppelin-solidity/contracts/utils/Strings.sol";

10: import "./openzeppelin-solidity/contracts/security/ReentrancyGuard.sol";

11: import "./lib/LiquidityAmounts.sol";

12: import "./lib/TickMath.sol";

13: import "../interfaces/IAaveOracle.sol";

14: import "../interfaces/IAaveOracle.sol";

```

```solidity
File: /contracts/helper/LPOracle.sol

4: import "../../interfaces/AggregatorV3Interface.sol";

```

```solidity
File: /contracts/helper/V3Proxy.sol

4: import "../openzeppelin-solidity/contracts/access/Ownable.sol";

5: import "../openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";

6: import "../openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";

7: import "../openzeppelin-solidity/contracts/security/ReentrancyGuard.sol";

```

#### Recommended Mitigation Steps:

`import {contract1 , contract2} from "filename.sol";` OR Use specific imports syntax per solidity docs recommendation.

### [NC-7] DON’T WORRY ABOUT KECCAK256 COLLISIONS

#### Description:

When cancelBid is called, the commitment is set to 0 in order to force to fail the computeCommitment comparison.In case of the result of computeCommitment returns 0, because a collision or an error, a cancelled bid will be processed as valid.It’s more resilient to use a different value as flag, b.pubKey will be safer and cheaper.

#### **Proof Of Concept**

```solidity
File: /contracts/RangeManager.sol

64:         continue;

```

### [NC-8] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

```solidity
File: /contracts/RangeManager.sol

2: pragma solidity ^0.8.0;

```

```solidity
File: /contracts/helper/FixedOracle.sol

1: pragma solidity ^0.8.0;

```

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-9] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

89:     treasury = _treasury;

92:     baseTokenIsToken0 = _baseTokenIsToken0;

102:     isEnabled = _isEnabled;

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

68:     token0 = _token0;

69:     token1 = _token1;

```

```solidity
File: /contracts/TokenisableRange.sol

117:     lowerTick = _lowerTick;

118:     upperTick = _upperTick;

```

```solidity
File: /contracts/helper/FixedOracle.sol

17:         hardcodedPrice = _hardcodedPrice;

25:         hardcodedPrice = _hardcodedPrice;

```

```solidity
File: /contracts/helper/V3Proxy.sol

76:         ROUTER = _router;

77:         QUOTER = _quoter;

```

### [NC-10] Omissions in events + Add parameter to event-emit

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

103:     emit SetEnabled(_isEnabled);

110:     emit SetTreasury(newTreasury);

131:     emit PushTick(tr);

160:     emit ShiftTick(tr);

186:     emit SetFee(newBaseFeeX4);

193:     emit SetTvlCap(newTvlCap);

362:     emit Rebalance(tickIndex);

```

```solidity
File: /contracts/RoeRouter.sol

50:     emit DeprecatePool(poolId);

86:     emit UpdateTreasury(newTreasury);

```

```solidity
File: /contracts/helper/FixedOracle.sol

26:         emit SetHardcodedPrice(_hardcodedPrice);

```

### [NC-11] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

313:   function removeFromAllTicks() internal {

321:   function removeFromTick(uint index) internal {

337:   function deployAssets() internal {

385:   function checkSetApprove(address token, address spender, uint amount) private {

404:   function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity){

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

67:   ) internal {

89:   ) internal {

130:     private returns (bool result)

257:     internal returns (uint debt)

336:   function checkExpectedBalances(address debtAsset, uint debtAmount, uint token0Amount, uint token1Amount) internal view

362:   ) internal returns (uint feeAmount) {

471:     internal returns (uint256 received)

491:   function swapTokensForExactTokens(IUniswapV2Router01 ammRouter, uint recvAmount, uint maxAmount, address[] memory path) internal {

514:     internal view returns (uint amountB)

533:   function sanityCheckUnderlying(address tr, address token0, address token1) internal {

544:   function addDust(address debtAsset, address token0, address token1) internal returns (uint amount){

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

64:     internal view

80:   function checkSetAllowance(address token, address spender, uint amount) internal {

89:   function cleanup(ILendingPool LP, address user, address asset) internal {

124:   function PMWithdraw(ILendingPool LP, address user, address asset, uint amount) internal {

138:   function validateValuesAgainstOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB, uint amountB) internal view {

```

```solidity
File: /contracts/RangeManager.sol

59:   function checkNewRange(uint128 start, uint128 end) internal view {

107:   function removeFromStep(uint256 step) internal {

151:   function transferAssetsIntoStep(TokenisableRange tr, uint256 step, uint256 amount0, uint256 amount1) internal {

190:   function cleanup() internal {

```

```solidity
File: /contracts/TokenisableRange.sol

50:   address private creator;

66:   function sqrt(uint x) internal pure returns (uint y) {

333:   function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {

```

```solidity
File: /contracts/helper/FixedOracle.sol

6:     int256 private hardcodedPrice;

7:     address private owner;

```

```solidity
File: /contracts/helper/LPOracle.sol

44:   function sqrt(uint x) internal pure returns (uint y) {

56:   function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {

```

### [NC-12] LINES ARE TOO LONG

#### Description:

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

[Reference](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length)

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

20: - Deposit one underlying asset split evenly into 2 or more consecutive ticks above/below the current price

21: - Withdraw one underlying asset, taken out evenly from 2 or more consecutive ticks

30:   event Deposit(address indexed sender, address indexed token, uint amount, uint liquidity);

31:   event Withdraw(address indexed sender, address indexed token, uint amount, uint liquidity);

83:     (address lpap, address _token0, address _token1,, ) = RoeRouter(roeRouter).pools(poolId);

87:     lendingPool = ILendingPool(ILendingPoolAddressesProvider(lpap).getLendingPool());

88:     oracle = IPriceOracle(ILendingPoolAddressesProvider(lpap).getPriceOracle());

125:         require( t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap");

127:         require( t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap");

146:         require( t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");

148:         require( t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");

214:   function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {

222:     amount = valueX8 * 10**ERC20(token).decimals() / oracle.getAssetPrice(token);

247:   function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity)

251:     require(token == address(token0) || token == address(token1), "GEV: Invalid Token");

268:     uint valueX8 = oracle.getAssetPrice(token) * (amount - fee) / 10**ERC20(token).decimals();

323:     address aTokenAddress = lendingPool.getReserveData(address(tr)).aTokenAddress;

343:     (uint amount0ft, uint amount1ft) = ticks[newTickIndex].getTokenAmountsExcludingFees(1e18);

376:     uint oraclePrice = 1e8 * oracle.getAssetPrice(address(token0)) / oracle.getAssetPrice(address(token1));

377:     if (oraclePrice < priceX8 * 101 / 100 && oraclePrice > priceX8 * 99 / 100) matches = true;

385:   function checkSetApprove(address token, address spender, uint amount) private {

386:     if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

404:   function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity){

422:     address aTokenAddress = lendingPool.getReserveData(address(t)).aTokenAddress;

431:       for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){

432:         (uint amt0, uint amt1) = ticks[activeTickIndex+1].getTokenAmountsExcludingFees(1e18);

433:         (uint amt0n, uint amt1n) = ticks[activeTickIndex+2].getTokenAmountsExcludingFees(1e18);

445:   function getAdjustedBaseFee(bool increaseToken0) public view returns (uint adjustedBaseFeeX4) {

447:     uint value0 = res0 * oracle.getAssetPrice(address(token0)) / 10**token0.decimals();

448:     uint value1 = res1 * oracle.getAssetPrice(address(token1)) / 10**token1.decimals();

457:     if (adjustedBaseFeeX4 > baseFeeX4 * 3 / 2) adjustedBaseFeeX4 = baseFeeX4 * 3 / 2;

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

8:   function getAmountsOut(uint256 amountIn, address[] calldata path) external returns (uint256[] memory amounts);

9:   function getAmountsIn(uint256 amountOut, address[] calldata path) external returns (uint256[] memory amounts);

16:   event BuyOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

17:   event SellOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

18:   event ClosePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

19:   event LiquidatePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);

20:   event ReducedPosition(address indexed user, address indexed asset, uint amount);

21:   event Swap(address indexed user, address sourceAsset, uint sourceAmount, address targetAsset, uint targetAmount);

48:       (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));

53:       (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));

68:     (ILendingPool lendingPool,,, address token0, address token1 ) = getPoolAddresses(poolId);

90:     (ILendingPool lendingPool,,, address token0, address token1) = getPoolAddresses(poolId);

92:     uint[2] memory amts = [ERC20(token0).balanceOf(address(this)), ERC20(token1).balanceOf(address(this))];

103:       uint debt = closeDebt(poolId, address(this), debtAsset, amount, collateral);

106:       emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1]);

132:     (, IPriceOracle oracle, IUniswapV2Router01 router, address token0, address token1) = getPoolAddresses(poolId);

135:     (uint256 amount0, uint256 amount1) = TokenisableRange(flashAsset).withdraw(flashAmount, 0, 0);

137:       require(sourceSwap == token0 || sourceSwap == token1, "OPM: Invalid Swap Token");

167:     require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");

176:     LP.flashLoan( address(this), options, amounts, flashtype, msg.sender, params, 0);

199:     bytes memory params = abi.encode(1, poolId, user, collateralAsset); // mode = 1 -> liquidation

200:     (ILendingPool LP,,, address token0, address token1) = getPoolAddresses(poolId);

207:     LP.flashLoan( address(this), options, amounts, flashtype, msg.sender, params, 0);

232:     (ILendingPool LP,,, address token0, address token1) = getPoolAddresses(poolId);

233:     uint debt = ERC20(LP.getReserveData(debtAsset).variableDebtTokenAddress).balanceOf(user);

259:     (ILendingPool LP,,IUniswapV2Router01 ammRouter, address token0, address token1) = getPoolAddresses(poolId);

261:     require(collateralAsset == token0 || collateralAsset == token1, "OPM: Invalid Collateral Asset");

270:       (uint token0Amount, uint token1Amount) = TokenisableRange(debtAsset).getTokenAmounts(debt);

276:         amtA = IERC20(LP.getReserveData(token0).aTokenAddress ).balanceOf(user);

277:         amtB = IERC20(LP.getReserveData(token1).aTokenAddress ).balanceOf(user);

282:           uint feeAmount = calculateAndSendFee(poolId, token0Amount, token1Amount, collateralAsset);

327:     if (user == msg.sender) swapTokens(poolId, collateralAsset == token0 ? token1 : token0, 0);

336:   function checkExpectedBalances(address debtAsset, uint debtAmount, uint token0Amount, uint token1Amount) internal view

341:     uint debtValue = TokenisableRange(debtAsset).latestAnswer() * debtAmount / 1e18;

342:     uint tokensValue = token0Amount * oracle.getAssetPrice(address(token0)) / 10**decimals0 + token1Amount * oracle.getAssetPrice(address(token1)) / 10**decimals1;

346:       || (tokensValue > debtValue * 98 / 100 && tokensValue < debtValue * 102 / 100),

363:     (, IPriceOracle oracle,, address token0, address token1) = getPoolAddresses(poolId);

365:     uint feeValueE8 = token0Amount * oracle.getAssetPrice(token0) / 10**ERC20(token0).decimals()

366:                     + token1Amount * oracle.getAssetPrice(token1) / 10**ERC20(token1).decimals() ;

367:     feeAmount = feeValueE8 * 10**ERC20(collateralAsset).decimals() / 100 / oracle.getAssetPrice(collateralAsset);

369:     require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral");

392:     (ILendingPool LP, IPriceOracle oracle,, address token0, address token1 ) = getPoolAddresses(poolId);

393:     require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

419:     (ILendingPool LP,,, address token0, address token1 ) = getPoolAddresses(poolId);

420:     require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

424:     (uint amount0, uint amount1) = TokenisableRange(optionAddress).withdraw(amount, 0, 0);

439:   function swapTokens(uint poolId, address sourceAsset, uint amount) public returns (uint received) {

440:     (ILendingPool LP, IPriceOracle oracle,IUniswapV2Router01 router, address token0, address token1) = getPoolAddresses(poolId);

441:     require(sourceAsset == token0 || sourceAsset == token1, "OPM: Invalid Swap Asset");

443:       amount = ERC20(LP.getReserveData(sourceAsset).aTokenAddress).balanceOf(msg.sender);

470:   function swapExactTokensForTokens(IUniswapV2Router01 ammRouter, IPriceOracle oracle, uint amount, address[] memory path)

473:     if (amount > 0 && AmountsRouter(address(ammRouter)).getAmountsOut(amount, path)[1] > 0){

477:         getTargetAmountFromOracle(oracle, path[0], amount, path[1]) * 99 / 100, // allow 1% slippage

491:   function swapTokensForExactTokens(IUniswapV2Router01 ammRouter, uint recvAmount, uint maxAmount, address[] memory path) internal {

493:     uint [] memory amountsIn = AmountsRouter(address(ammRouter)).getAmountsIn(recvAmount, path);

495:     require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );

496:     require( amountsIn[0] <= ERC20(path[0]).balanceOf(address(this)), "OPM: Insufficient Token Amount" );

513:   function getTargetAmountFromOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB)

517:       uint valueA = amountA * oracle.getAssetPrice(assetA) / 10**ERC20(assetA).decimals();

518:       uint valueB = amountB * oracle.getAssetPrice(assetB) / 10**ERC20(assetB).decimals();

524:     amountB = amountA * priceAssetA * 10**ERC20(assetB).decimals() / 10**ERC20(assetA).decimals() / priceAssetB;

533:   function sanityCheckUnderlying(address tr, address token0, address token1) internal {

536:     require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");

544:   function addDust(address debtAsset, address token0, address token1) internal returns (uint amount){

546:     uint scale0 = 10**(20 - ERC20(token0).decimals()) * oracle.getAssetPrice(token0) / 1e8;

547:     uint scale1 = 10**(20 - ERC20(token1).decimals()) * oracle.getAssetPrice(token1) / 1e8;

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

7: import {ILendingPoolAddressesProvider} from "../../interfaces/ILendingPoolAddressesProvider.sol";

21:   ILendingPoolAddressesProvider public ADDRESSES_PROVIDER; // IFlashLoanReceiver  requirement

65:     returns( ILendingPool lp, IPriceOracle oracle, IUniswapV2Router01 router, address token0, address token1)

67:     (address lpap, address _token0, address _token1, address r, ) = ROEROUTER.pools(poolId);

71:     oracle = IPriceOracle(ILendingPoolAddressesProvider(lpap).getPriceOracle());

80:   function checkSetAllowance(address token, address spender, uint amount) internal {

81:     if ( ERC20(token).allowance(address(this), spender) < amount ) ERC20(token).safeIncreaseAllowance(spender, type(uint256).max);

95:       uint debt = ERC20(LP.getReserveData(asset).variableDebtTokenAddress).balanceOf(user);

115:     ERC20(asset).safeTransfer(ROEROUTER.treasury(), ERC20(asset).balanceOf(address(this)));

124:   function PMWithdraw(ILendingPool LP, address user, address asset, uint amount) internal {

138:   function validateValuesAgainstOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB, uint amountB) internal view {

144:     else if (decimalsA < decimalsB) valueB = valueB / 10 ** (decimalsB - decimalsA);

```

```solidity
File: /contracts/RangeManager.sol

36:   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);

75:   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

83:     IAaveOracle oracle = IAaveOracle(ILendingPoolAddressesProvider( LENDING_POOL.getAddressesProvider() ).getPriceOracle());

85:     tokenisedRanges[ tokenisedRanges.length - 1 ].initProxy(oracle, ASSET_0, ASSET_1, startX10, endX10, startName, endName, false);

86:     tokenisedTicker[ tokenisedTicker.length - 1 ].initProxy(oracle, ASSET_0, ASSET_1, startX10, endX10, startName, endName, true);

95:   function initRange(address tr, uint amount0, uint amount1) external onlyOwner {

101:     ERC20(tr).safeTransfer(msg.sender, TokenisableRange(tr).balanceOf(address(this)));

108:     require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");

111:     trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress).balanceOf(msg.sender);

114:           LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress,

118:         trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));

123:     trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress).balanceOf(msg.sender);

126:           LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress,

130:         uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));

151:   function transferAssetsIntoStep(TokenisableRange tr, uint256 step, uint256 amount0, uint256 amount1) internal {

154:       LENDING_POOL.PMTransfer( LENDING_POOL.getReserveData(address(ASSET_0)).aTokenAddress, msg.sender, amount0 );

159:       LENDING_POOL.PMTransfer( LENDING_POOL.getReserveData(address(ASSET_1)).aTokenAddress, msg.sender, amount1 );

175:   function transferAssetsIntoRangerStep(uint256 step, uint256 amount0, uint256 amount1) nonReentrant external {

184:   function transferAssetsIntoTickerStep(uint256 step, uint256 amount0, uint256 amount1) nonReentrant external {

```

```solidity
File: /contracts/RoeRouter.sol

76:     pools.push(RoePool(lendingPoolAddressProvider, token0, token1, ammRouter, false));

```

```solidity
File: /contracts/TokenisableRange.sol

21:   event InitTR(address asset0, address asset1, uint128 startX10, uint128 endX10);

54:   address public TREASURY_DEPRECATED = 0x22Cc3f665ba4C898226353B672c5123c58751692;

58:   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);

59:   IUniswapV3Factory constant public V3_FACTORY = IUniswapV3Factory(0x1F98431c8aD98523631AE4a59f267346ea31F984);

60:   address constant public treasury = 0x22Cc3f665ba4C898226353B672c5123c58751692;

85:   function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {

99:     int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );

100:     int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );

106:       _upperTick = (midleTick + int24(feeTier)) - (midleTick + int24(feeTier)) % int24(feeTier * 2);

108:       _name     = string(abi.encodePacked("Ticker ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));

109:      _symbol    = string(abi.encodePacked("T-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));

112:       _lowerTick = (_lowerTick + int24(feeTier)) - (_lowerTick + int24(feeTier)) % int24(feeTier * 2);

113:       _upperTick = (_upperTick + int24(feeTier)) - (_upperTick + int24(feeTier)) % int24(feeTier * 2);

114:       _name     = string(abi.encodePacked("Ranger ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));

115:       _symbol   = string(abi.encodePacked("R-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));

125:   function name()     public view virtual override returns (string memory) { return _name; }

127:   function symbol()   public view virtual override returns (string memory) { return _symbol; }

159:     TOKEN0.token.safeTransfer( msg.sender,  TOKEN0.token.balanceOf(address(this)));

160:     TOKEN1.token.safeTransfer(msg.sender, TOKEN1.token.balanceOf(address(this)));

193:       (uint128 newLiquidity, uint256 added0, uint256 added1) = POS_MGR.increaseLiquidity(

206:       uint addedValue = added0 * token0Price / 10**TOKEN0.decimals + added1 * token1Price / 10**TOKEN1.decimals;

207:       uint totalValue =   bal0 * token0Price / 10**TOKEN0.decimals +   bal1 * token1Price / 10**TOKEN1.decimals;

209:       require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

222:   function deposit(uint256 n0, uint256 n1) external nonReentrant returns (uint256 lpAmt) {

238:       address pool = V3_FACTORY.getPool(address(TOKEN0.token), address(TOKEN1.token), feeTier * 100);

240:       (uint256 token0Amount, uint256 token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick), liquidity);

253:     (uint128 newLiquidity, uint256 added0, uint256 added1) = POS_MGR.increaseLiquidity(

274:       feeLiquidity = newLiquidity * ( (fee0 * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (fee1 * TOKEN1_PRICE / 10 ** TOKEN1.decimals) )

275:                                     / ( (added0   * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (added1   * TOKEN1_PRICE / 10 ** TOKEN1.decimals) );

292:   function withdraw(uint256 lp, uint256 amount0Min, uint256 amount1Min) external nonReentrant returns (uint256 removed0, uint256 removed1) {

333:   function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {

335:     if (TOKEN0_PRICE == 0) TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token));

336:     if (TOKEN1_PRICE == 0) TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token));

338:     (amt0, amt1) = LiquidityAmounts.getAmountsForLiquidity( uint160( sqrt( (2 ** 192 * ((TOKEN0_PRICE * 10 ** TOKEN1.decimals) / TOKEN1_PRICE)) / ( 10 ** TOKEN0.decimals ) ) ), TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  liquidity);

343:   function returnExpectedBalance(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 amt0, uint256 amt1) {

344:     (amt0, amt1) = returnExpectedBalanceWithoutFees(TOKEN0_PRICE, TOKEN1_PRICE);

352:   function getValuePerLPAtPrice(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 priceX1e8) {

354:     (uint256 amt0, uint256 amt1) = returnExpectedBalance(TOKEN0_PRICE, TOKEN1_PRICE);

355:     uint totalValue = TOKEN0_PRICE * amt0 / (10 ** TOKEN0.decimals) + amt1 * TOKEN1_PRICE / (10 ** TOKEN1.decimals);

362:     return getValuePerLPAtPrice(ORACLE.getAssetPrice(address(TOKEN0.token)), ORACLE.getAssetPrice(address(TOKEN1.token)));

368:   function getTokenAmountsExcludingFees(uint amount) public view returns (uint token0Amount, uint token1Amount){

369:     address pool = V3_FACTORY.getPool(address(TOKEN0.token), address(TOKEN1.token), feeTier * 100);

371:     (token0Amount, token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  uint128 ( uint(liquidity) * amount / totalSupply() ) );

377:   function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){

```

```solidity
File: /contracts/helper/LPOracle.sol

8:   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

29:     require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");

56:   function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {

77:       norm_a, norm_b represents a, b adjusted along the k curve such that it represents the amounts the uniswap pool will contain at the Chainlink oracle price

97:       The normalised positions (18 decimals) are multiplied with the chainlink value (8 decimals), giving val.

98:       val is divided by LP_TOKEN.totalSupply(), which has 18 decimals, and casted to an int

102:     uint val = norm_a * priceA * 10**(18-decimalsA) + norm_b * 10**(18-decimalsB) * priceB;

```

```solidity
File: /contracts/helper/V3Proxy.sol

20:     function exactInputSingle(ExactInputSingleParams calldata params) external payable returns (uint256 amountOut);

32:     function exactOutputSingle(ExactOutputSingleParams calldata params) external payable returns (uint256 amountIn);

95:         token.safeTransfer(msg.sender, token.balanceOf( address(this) ) );  // msg.sender has been Required to be owner

98:     function getAmountsOut(uint amountIn, address[] calldata path) external returns (uint[] memory amounts) {

102:         amounts[1] = QUOTER.quoteExactInputSingle(path[0], path[1], feeTier, amountIn, 0);

105:     function getAmountsIn(uint amountOut, address[] calldata path) external returns (uint[] memory amounts) {

108:         amounts[0] = QUOTER.quoteExactOutputSingle(path[0], path[1], feeTier, amountOut, 0);

112:     function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

119:         amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountIn, amountOutMin, 0));

124:     function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

130:         amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountOut, amountInMax, 0));

137:     function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

142:         amounts[1] = ROUTER.exactInputSingle{value: msg.value}(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, msg.value, amountOutMin, 0));

147:     function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

151:         amounts[0] = ROUTER.exactOutputSingle{value: msg.value}(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountOut, msg.value, 0));

160:     function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

167:         amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountOut, amountInMax, 0));

178:     function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

186:         amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountIn, amountOutMin, 0));

193:         emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);

```

### [NC-13] Constants should be defined rather than using magic numbers

#### **Proof Of Concept**

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

546:     uint scale0 = 10**(20 - ERC20(token0).decimals()) * oracle.getAssetPrice(token0) / 1e8;

547:     uint scale1 = 10**(20 - ERC20(token1).decimals()) * oracle.getAssetPrice(token1) / 1e8;

```

```solidity
File: /contracts/helper/LPOracle.sol

97:       The normalised positions (18 decimals) are multiplied with the chainlink value (8 decimals), giving val.

102:     uint val = norm_a * priceA * 10**(18-decimalsA) + norm_b * 10**(18-decimalsB) * priceB;

```

## Low Issues

|      | Issue                                                 |
| ---- | :---------------------------------------------------- |
| L-1  | Avoid `transfer()`/`send()` as reentrancy mitigations |
| L-2  | Do not use deprecated library functions               |
| L-3  | DON’T USE OWNER AND TIMELOCK                          |
| L-4  | Initializers could be front-run                       |
| L-5  | LOSS OF PRECISION DUE TO ROUNDING                     |
| L-6  | Unify return criteria                                 |
| L-7  | OWNER CAN RENOUNCE OWNERSHIP                          |
| L-8  | Timestamp Dependence                                  |
| L-9  | Account existence check for low-level calls           |
| L-10 | Unsafe ERC20 operation(s)                             |

### [L-1] Avoid `transfer()`/`send()` as reentrancy mitigations

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

232:       payable(msg.sender).transfer(bal);

```

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

### [L-2] Do not use deprecated library functions

#### Description:

You use safeApprove of openZeppelin although it’s deprecated. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/566a774222707e424896c0c390a84dc3c13bdcb2/contracts/token/ERC20/utils/SafeERC20.sol#L38%3E)).

You should change it to increase/decrease Allowance as OpenZeppilin says.

#### **Proof Of Concept**

```solidity
File: /contracts/helper/V3Proxy.sol

116:         ogInAsset.safeApprove(address(ROUTER), amountIn);

120:         ogInAsset.safeApprove(address(ROUTER), 0);

128:         ogInAsset.safeApprove(address(ROUTER), amountInMax);

133:         ogInAsset.safeApprove(address(ROUTER), 0);

165:         ogInAsset.safeApprove(address(ROUTER), amountInMax);

169:         ogInAsset.safeApprove(address(ROUTER), 0);

183:         ogInAsset.safeApprove(address(ROUTER), amountIn);

187:         ogInAsset.safeApprove(address(ROUTER), 0);

```

### [L-3] DON’T USE OWNER AND TIMELOCK

#### Description:

Using a timelock contract gives confidence to the user, but when check onlyByOwnGov allow the owner and the timelock The owner manipulates the contract without a lock time period.

#### **Proof Of Concept**

```solidity
File: /contracts/helper/FixedOracle.sol

11:         require(msg.sender == owner, "Only the owner can call this function.");

```

### [L-4] Initializers could be front-run

#### Description:

Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

#### **Proof Of Concept**

```solidity
File: /contracts/RangeManager.sol

100:     TokenisableRange(tr).init(amount0, amount1);

```

```solidity
File: /contracts/TokenisableRange.sol

134:   function init(uint n0, uint n1) external {

```

### [L-5] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

221:     uint valueX8 = vaultValueX8 * liquidity / totalSupply();

222:     amount = valueX8 * 10**ERC20(token).decimals() / oracle.getAssetPrice(token);

266:     uint fee = amount * getAdjustedBaseFee(token == address(token0)) / 1e4;

268:     uint valueX8 = oracle.getAssetPrice(token) * (amount - fee) / 10**ERC20(token).decimals();

277:       liquidity = tSupply * valueX8 / vaultValueX8;

293:     priceX8 = vaultValue * 1e18 / supply;

374:     priceX8 = priceX8 * uint(sqrtPriceX96 / 2 ** 12) ** 2 * 1e8 / 2**168;

376:     uint oraclePrice = 1e8 * oracle.getAssetPrice(address(token0)) / oracle.getAssetPrice(address(token1));

377:     if (oraclePrice < priceX8 * 101 / 100 && oraclePrice > priceX8 * 99 / 100) matches = true;

396:       valueX8 += bal * t.latestAnswer() / 1e18;

448:     uint value1 = res1 * oracle.getAssetPrice(address(token1)) / 10**token1.decimals();

451:       adjustedBaseFeeX4 = baseFeeX4 * value0 / (value1 + 1);

457:     if (adjustedBaseFeeX4 > baseFeeX4 * 3 / 2) adjustedBaseFeeX4 = baseFeeX4 * 3 / 2;

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

341:     uint debtValue = TokenisableRange(debtAsset).latestAnswer() * debtAmount / 1e18;

342:     uint tokensValue = token0Amount * oracle.getAssetPrice(address(token0)) / 10**decimals0 + token1Amount * oracle.getAssetPrice(address(token1)) / 10**decimals1;

346:       || (tokensValue > debtValue * 98 / 100 && tokensValue < debtValue * 102 / 100),

365:     uint feeValueE8 = token0Amount * oracle.getAssetPrice(token0) / 10**ERC20(token0).decimals()

366:                     + token1Amount * oracle.getAssetPrice(token1) / 10**ERC20(token1).decimals() ;

367:     feeAmount = feeValueE8 * 10**ERC20(collateralAsset).decimals() / 100 / oracle.getAssetPrice(collateralAsset);

477:         getTargetAmountFromOracle(oracle, path[0], amount, path[1]) * 99 / 100, // allow 1% slippage

517:       uint valueA = amountA * oracle.getAssetPrice(assetA) / 10**ERC20(assetA).decimals();

518:       uint valueB = amountB * oracle.getAssetPrice(assetB) / 10**ERC20(assetB).decimals();

524:     amountB = amountA * priceAssetA * 10**ERC20(assetB).decimals() / 10**ERC20(assetA).decimals() / priceAssetB;

546:     uint scale0 = 10**(20 - ERC20(token0).decimals()) * oracle.getAssetPrice(token0) / 1e8;

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

145:     require( valueA <= valueB * 101 / 100, "PM: LP Oracle Error");

146:     require( valueB <= valueA * 101 / 100, "PM: LP Oracle Error");

```

```solidity
File: /contracts/TokenisableRange.sol

99:     int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );

100:     int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );

151:          amount0Min: n0 * 95 / 100,

152:          amount1Min: n1 * 95 / 100,

178:     uint tf0 = newFee0 * treasuryFee / 100;

206:       uint addedValue = added0 * token0Price / 10**TOKEN0.decimals + added1 * token1Price / 10**TOKEN1.decimals;

207:       uint totalValue =   bal0 * token0Price / 10**TOKEN0.decimals +   bal1 * token1Price / 10**TOKEN1.decimals;

208:       uint liquidityValue = totalValue * newLiquidity / liquidity;

209:       require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

242:       if (token1Amount + fee1 > 0) newFee1 = n1 * fee1 / (token1Amount + fee1);

258:         amount0Min: n0 * 95 / 100,

274:       feeLiquidity = newLiquidity * ( (fee0 * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (fee1 * TOKEN1_PRICE / 10 ** TOKEN1.decimals) )

275:                                     / ( (added0   * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (added1   * TOKEN1_PRICE / 10 ** TOKEN1.decimals) );

278:     lpAmt = totalSupply() * newLiquidity / (liquidity + feeLiquidity);

294:     uint removedLiquidity = uint(liquidity) * lp / totalSupply();

316:       TOKEN0.token.safeTransfer( msg.sender, fee0 * lp / totalSupply());

317:       removed0 += fee0 * lp / totalSupply();

318:       fee0 -= fee0 * lp / totalSupply();

321:       TOKEN1.token.safeTransfer(  msg.sender, fee1 * lp / totalSupply());

322:       removed1 += fee1 * lp / totalSupply();

323:       fee1 -= fee1 * lp / totalSupply();

338:     (amt0, amt1) = LiquidityAmounts.getAmountsForLiquidity( uint160( sqrt( (2 ** 192 * ((TOKEN0_PRICE * 10 ** TOKEN1.decimals) / TOKEN1_PRICE)) / ( 10 ** TOKEN0.decimals ) ) ), TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  liquidity);

355:     uint totalValue = TOKEN0_PRICE * amt0 / (10 ** TOKEN0.decimals) + amt1 * TOKEN1_PRICE / (10 ** TOKEN1.decimals);

356:     return totalValue * 1e18 / totalSupply();

371:     (token0Amount, token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  uint128 ( uint(liquidity) * amount / totalSupply() ) );

379:     token0Amount += fee0 * amount / totalSupply();

```

```solidity
File: /contracts/helper/LPOracle.sol

92:       norm_b = sqrt( a * b * priceA / 10**(decimalsA-decimalsB) / priceB );

94:     uint norm_a = a * b / norm_b;

```

### [L-6] Unify return criteria

#### Description:

In contracts, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

177:   function getTickLength() public view returns(uint len){

214:   function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {

247:   function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity)

289:   function latestAnswer() external view returns (uint256 priceX8) {

298:   function getReserves() public view returns (uint amount0, uint amount1){

367:   function poolMatchesOracle() public view returns (bool matches){

392:   function getTVL() public view returns (uint valueX8){

404:   function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity){

420:   function getTickBalance(uint index) public view returns (uint liquidity) {

428:   function getActiveTickIndex() public view returns (uint activeTickIndex) {

445:   function getAdjustedBaseFee(bool increaseToken0) public view returns (uint adjustedBaseFeeX4) {

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

8:   function getAmountsOut(uint256 amountIn, address[] calldata path) external returns (uint256[] memory amounts);

9:   function getAmountsIn(uint256 amountOut, address[] calldata path) external returns (uint256[] memory amounts);

44:   ) override external returns (bool result) {

130:     private returns (bool result)

257:     internal returns (uint debt)

362:   ) internal returns (uint feeAmount) {

439:   function swapTokens(uint poolId, address sourceAsset, uint amount) public returns (uint received) {

471:     internal returns (uint256 received)

514:     internal view returns (uint amountB)

544:   function addDust(address debtAsset, address token0, address token1) internal returns (uint amount){

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

52:   ) virtual external returns (bool result) {

65:     returns( ILendingPool lp, IPriceOracle oracle, IUniswapV2Router01 router, address token0, address token1)

```

```solidity
File: /contracts/RangeManager.sol

212:   function getStepListLength() external view returns (uint256 listLength) {

```

```solidity
File: /contracts/RoeRouter.sol

40:   function getPoolsLength() public view returns (uint poolLength) {

66:     returns (uint poolId)

```

```solidity
File: /contracts/TokenisableRange.sol

66:   function sqrt(uint x) internal pure returns (uint y) {

125:   function name()     public view virtual override returns (string memory) { return _name; }

127:   function symbol()   public view virtual override returns (string memory) { return _symbol; }

222:   function deposit(uint256 n0, uint256 n1) external nonReentrant returns (uint256 lpAmt) {

292:   function withdraw(uint256 lp, uint256 amount0Min, uint256 amount1Min) external nonReentrant returns (uint256 removed0, uint256 removed1) {

333:   function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {

343:   function returnExpectedBalance(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 amt0, uint256 amt1) {

352:   function getValuePerLPAtPrice(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 priceX1e8) {

361:   function latestAnswer() public view returns (uint256 priceX1e8) {

368:   function getTokenAmountsExcludingFees(uint amount) public view returns (uint token0Amount, uint token1Amount){

377:   function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){

```

```solidity
File: /contracts/helper/FixedOracle.sol

20:     function latestAnswer() external view returns (int256) {

32:         returns (

```

```solidity
File: /contracts/helper/LPOracle.sol

7:   function totalSupply() external view returns (uint);

8:   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

9:   function token0() external view returns (address);

10:   function token1() external view returns (address);

14:   function decimals() external view returns (uint8);

38:   function decimals() external pure returns (uint8) {

44:   function sqrt(uint x) internal pure returns (uint y) {

56:   function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {

69:   function latestAnswer() external view returns (int256) {

```

```solidity
File: /contracts/helper/V3Proxy.sol

20:     function exactInputSingle(ExactInputSingleParams calldata params) external payable returns (uint256 amountOut);

32:     function exactOutputSingle(ExactOutputSingleParams calldata params) external payable returns (uint256 amountIn);

34:     function WETH9() external view returns (address);

45:     ) external returns (uint256 amountOut);

53:     ) external returns (uint256 amountIn);

82:     function WETH() external view returns (address) {

98:     function getAmountsOut(uint amountIn, address[] calldata path) external returns (uint[] memory amounts) {

105:     function getAmountsIn(uint amountOut, address[] calldata path) external returns (uint[] memory amounts) {

112:     function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

124:     function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {

137:     function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

147:     function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

160:     function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

178:     function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {

```

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

### [L-7] OWNER CAN RENOUNCE OWNERSHIP

#### Description:

Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements `renounceOwnership`. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

4: import "./openzeppelin-solidity/contracts/access/Ownable.sol";

27: contract GeVault is ERC20, Ownable, ReentrancyGuard {

```

```solidity
File: /contracts/RangeManager.sol

20: import "./openzeppelin-solidity/contracts/access/Ownable.sol";

25: contract RangeManager is ReentrancyGuard, Ownable {

```

```solidity
File: /contracts/RoeRouter.sol

4: import "./openzeppelin-solidity/contracts/access/Ownable.sol";

10: contract RoeRouter is Ownable {

```

```solidity
File: /contracts/helper/V3Proxy.sol

4: import "../openzeppelin-solidity/contracts/access/Ownable.sol";

60: contract V3Proxy is ReentrancyGuard, Ownable {

```

#### Recommended Mitigation Steps:

We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.`

### [L-8] A SINGLE POINT OF FAILURE

#### Description:

The owner role has a single point of failure and onlyOwner can use critical a few functions.

Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

101:   function setEnabled(bool _isEnabled) public onlyOwner {

108:   function setTreasury(address newTreasury) public onlyOwner {

116:   function pushTick(address tr) public onlyOwner {

137:   function shiftTick(address tr) public onlyOwner {

167:   function modifyTick(address tr, uint index) public onlyOwner {

183:   function setBaseFee(uint newBaseFeeX4) public onlyOwner {

191:   function setTvlCap(uint newTvlCap) public onlyOwner {

```

```solidity
File: /contracts/RangeManager.sol

75:   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

95:   function initRange(address tr, uint amount0, uint amount1) external onlyOwner {

```

```solidity
File: /contracts/RoeRouter.sol

48:   function deprecatePool(uint poolId) public onlyOwner {

65:     public onlyOwner

83:   function setTreasury(address newTreasury) public onlyOwner {

```

```solidity
File: /contracts/helper/FixedOracle.sol

10:     modifier onlyOwner() {

24:     function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {

```

```solidity
File: /contracts/helper/V3Proxy.sol

94:     function emergencyWithdraw(ERC20 token) onlyOwner external {

```

#### Recommended Mitigation Steps:

Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Also detail them in documentation and NatSpec comments.

### [L-8] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

As for block.number, considering the block time on Ethereum is generally about 14 seconds, it`s possible to predict the time delta between blocks. However, block times are not constant and are subject to change for a variety of reasons, e.g. fork reorganisations and the difficulty bomb. Due to variable block times, block.number should also not be relied on for precise calculations of time.
Reference: https://swcregistry.io/docs/SWC-116
[code4arena](https://code4rena.com/reports/2022-10-holograph/#18-use-of-blocktimestamp)
Reference: (https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md)

#### **Proof Of Concept**

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

480:         block.timestamp

502:       block.timestamp

```

```solidity
File: /contracts/TokenisableRange.sol

154:          deadline: block.timestamp

200:           deadline: block.timestamp

260:         deadline: block.timestamp

301:         deadline: block.timestamp

```

```solidity
File: /contracts/helper/FixedOracle.sol

40:         roundId = uint80(block.number);

42:         startedAt = block.timestamp;

43:         updatedAt = block.timestamp;

```

### [L-9] Account existence check for low-level calls

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-callsn

#### **Proof Of Concept**

```solidity
File: /contracts/TokenisableRange.sol

136:     require(msg.sender == creator, "Unallowed call");

```

```solidity
File: /contracts/helper/FixedOracle.sol

11:         require(msg.sender == owner, "Only the owner can call this function.");

```

```solidity
File: /contracts/helper/V3Proxy.sol

156:         msg.sender.call{value: msg.value - amounts[0]}("");

174:         payable(msg.sender).call{value: amountOut}("");

192:         payable(msg.sender).call{value: amounts[1]}("");

```

#### Recommended Mitigation Steps:

In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

### [L-10] Unsafe ERC20 operation(s)

#### Description:

There are ERC20 tokens that charge fee for every transfer() / transferFrom().

Vault.sol#addValue() assumes that the received amount is the same as the transfer amount, and uses it to calculate attributions, balance amounts, etc.

But, the actual transferred amount can be lower for those tokens. Therefore it’s recommended to use the balance change before and after the transfer instead of the amount.

This way you also support the tokens with transfer fee - that are popular.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

232:       payable(msg.sender).transfer(bal);

```

```solidity
File: /contracts/RangeManager.sol

165:     tr.approve(address(LENDING_POOL), lpAmt);

```

```solidity
File: /contracts/TokenisableRange.sol

228:     TOKEN0.token.transferFrom(msg.sender, address(this), n0);

229:     TOKEN1.token.transferFrom(msg.sender, address(this), n1);

```
