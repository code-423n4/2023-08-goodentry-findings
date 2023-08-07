## Gas Optimizations

|        | Issue                                                                               |
| ------ | :---------------------------------------------------------------------------------- |
| GAS-1  | MISSING DEADLINE CHECKS ALLOW PENDING TRANSACTIONS TO BE MALICIOUSLY EXECUTED       |
| GAS-2  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE         |
| GAS-3  | Using bools for storage incurs overhead                                             |
| GAS-4  | Setting the constructor to payable                                                  |
| GAS-5  | DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION |
| GAS-6  | USE FUNCTION INSTEAD OF MODIFIERS                                                   |
| GAS-7  | require()/revert() strings longer than 32 bytes cost extra gas                      |
| GAS-8  | USING PRIVATE RATHER THAN PUBLIC FOR CONSTANTS, SAVES GAS                           |
| GAS-9  | MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL                        |
| GAS-10 | Functions guaranteed to revert when called by normal users can be marked payable    |
| GAS-11 | Optimize names to save gas                                                          |
| GAS-12 | The increment in for loop postcondition can be made unchecked                       |
| GAS-13 | Using `private` rather than `public` for constants, saves gas                       |
| GAS-14 | Use shift Right/Left instead of division/multiplication if possible                 |
| GAS-15 | Splitting require() statements that use && saves gas                                |
| GAS-16 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                 |
| GAS-17 | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead              |
| GAS-18 | USE BYTES32 INSTEAD OF STRING                                                       |
| GAS-19 | Can make the variable outside the loop to save gas                                  |

### [GAS-1] MISSING DEADLINE CHECKS ALLOW PENDING TRANSACTIONS TO BE MALICIOUSLY EXECUTED

#### Description:

AMMs should provide their users with an option to limit the execution of their pending actions, such as swaps or adding and removing liquidity. The most common solution is to include a deadline timestamp as a parameter (for example see Uniswap V2). If such an option is not present, users can unknowingly perform bad trades:

#### **Proof Of Concept**

```solidity
File: /contracts/helper/LPOracle.sol

76:       In the uniswap AMM model, a*b is always a constant k (ignoring fees)

```

### [GAS-2] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

232:       payable(msg.sender).transfer(bal);

```

```solidity
File: /contracts/RangeManager.sol

151:   function transferAssetsIntoStep(TokenisableRange tr, uint256 step, uint256 amount0, uint256 amount1) internal {

175:   function transferAssetsIntoRangerStep(uint256 step, uint256 amount0, uint256 amount1) nonReentrant external {

176:     transferAssetsIntoStep(tokenisedRanges[step], step, amount0, amount1);

184:   function transferAssetsIntoTickerStep(uint256 step, uint256 amount0, uint256 amount1) nonReentrant external {

185:     transferAssetsIntoStep(tokenisedTicker[step], step, amount0, amount1);

```

```solidity
File: /contracts/TokenisableRange.sol

228:     TOKEN0.token.transferFrom(msg.sender, address(this), n0);

229:     TOKEN1.token.transferFrom(msg.sender, address(this), n1);

```

### [GAS-3] Using bools for storage incurs overhead

#### Description:

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

50:   bool public isEnabled = true;

64:   bool public baseTokenIsToken0;

```

### [GAS-4] Setting the constructor to payable

#### Description:

Saves ~13 gas per instance

[Code4arena example](https://code4rena.com/reports/2022-12-caviar/#g-10-setting-the-constructor-to-payable)

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

67:   constructor(

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

25:   constructor (address roerouter) PositionManager(roerouter) {}

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

29:   constructor(address roerouter) {

```

```solidity
File: /contracts/RangeManager.sol

48:   constructor(ILendingPool lendingPool, ERC20 _asset0, ERC20 _asset1)  {

```

```solidity
File: /contracts/RoeRouter.sol

33:   constructor (address treasury_) {

```

```solidity
File: /contracts/helper/FixedOracle.sol

15:     constructor(int256 _hardcodedPrice) {

```

```solidity
File: /contracts/helper/LPOracle.sol

28: 	constructor (address lpToken, address clToken0, address clToken1 ){

```

```solidity
File: /contracts/helper/V3Proxy.sol

75:     constructor(ISwapRouter _router, IQuoter _quoter, uint24 _fee) {

```

### [GAS-5] DUPLICATED REQUIRE()/REVERT() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION

#### **Proof Of Concept**

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

69:     require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");

91:     require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");

393:     require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

420:     require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

```

```solidity
File: /contracts/helper/V3Proxy.sol

87:         require(acceptPayable, "CannotReceiveETH");

91:        require(acceptPayable, "CannotReceiveETH");

99:         require(path.length == 2, "Direct swap only");

106:         require(path.length == 2, "Direct swap only");

113:         require(path.length == 2, "Direct swap only");

125:         require(path.length == 2, "Direct swap only");

138:         require(path.length == 2, "Direct swap only");

148:         require(path.length == 2, "Direct swap only");

149:         require(path[0] == ROUTER.WETH9(), "Invalid path");

161:         require(path.length == 2, "Direct swap only");

162:         require(path[1] == ROUTER.WETH9(), "Invalid path");

179:         require(path.length == 2, "Direct swap only");

180:         require(path[1] == ROUTER.WETH9(), "Invalid path");

```

### [GAS-6] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: /contracts/helper/FixedOracle.sol

10:     modifier onlyOwner() {

```

### [GAS-7] require()/revert() strings longer than 32 bytes cost extra gas

#### Description:

Each extra memory word of bytes past the original 32 incurs an MSTORE which costs 3 gas.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: /contracts/helper/FixedOracle.sol

11:         require(msg.sender == owner, "Only the owner can call this function.");

```

### [GAS-8] USING PRIVATE RATHER THAN PUBLIC FOR CONSTANTS, SAVES GAS

#### Description:

When constants are marked public, extra getter functions are created, increasing the deployment cost. Marking these functions private will decrease gas cost. One can still read these variables through the source code. If they need to be accessed by an external contract, a separate single getter function can be used to return all constants as a tuple. There are four instances of public constants.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

62:   uint public constant nearbyRanges = 2;

```

### [GAS-9] MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL

#### Description:

https://code4rena.com/reports/2022-09-y2k-finance#g-03-modifiers-are-redundant-if-used-only-once-or-not-used-at-all-5-instances

#### **Proof Of Concept**

```solidity
File: /contracts/helper/FixedOracle.sol

10:     modifier onlyOwner() {

```

### [GAS-10] Functions guaranteed to revert when called by normal users can be marked payable

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

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

99:         require(path.length == 2, "Direct swap only");

106:         require(path.length == 2, "Direct swap only");

113:         require(path.length == 2, "Direct swap only");

125:         require(path.length == 2, "Direct swap only");

138:         require(path.length == 2, "Direct swap only");

148:         require(path.length == 2, "Direct swap only");

161:         require(path.length == 2, "Direct swap only");

179:         require(path.length == 2, "Direct swap only");

```

### [GAS-11] Optimize names to save gas

#### Description:

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

[Solidity optimize name github](https://github.com/enzosv/solidity-optimize-name)

[Source](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

27: contract GeVault is ERC20, Ownable, ReentrancyGuard {

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

7: interface  AmountsRouter {

12: contract OptionsPositionManager is PositionManager {

```

```solidity
File: /contracts/PositionManager/PositionManager.sol

17: contract PositionManager is IFlashLoanReceiver {

```

```solidity
File: /contracts/RangeManager.sol

25: contract RangeManager is ReentrancyGuard, Ownable {

```

```solidity
File: /contracts/RoeRouter.sol

10: contract RoeRouter is Ownable {

```

```solidity
File: /contracts/TokenisableRange.sol

18: contract TokenisableRange is ERC20("", ""), ReentrancyGuard {

```

```solidity
File: /contracts/helper/FixedOracle.sol

3: contract HardcodedPriceOracle {

```

```solidity
File: /contracts/helper/LPOracle.sol

6: interface UniswapV2Pair {

13: interface IERC20 {

17: contract LPOracle {

```

```solidity
File: /contracts/helper/V3Proxy.sol

9: interface ISwapRouter {

38: interface IQuoter {

56: interface IWETH9 is IERC20 {

60: contract V3Proxy is ReentrancyGuard, Ownable {

```

### [GAS-12] The increment in for loop postcondition can be made unchecked

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.
the for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks.One can manually do this.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

154:         for (uint k = 0; k < ticks.length - 2; k++)

299:     for (uint k = 0; k < ticks.length; k++){

314:     for (uint k = 0; k < ticks.length; k++){

393:     for(uint k=0; k<ticks.length; k++){

431:       for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

71:     for ( uint8 k = 0; k<assets.length; k++){

93:     for ( uint8 k =0; k<assets.length; k++){

```

```solidity
File: /contracts/RangeManager.sol

62:     for (uint i = 0; i < len; i++) {

```

### [GAS-13] Using `private` rather than `public` for constants, saves gas

#### Description:

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves **3406-3606 gas** in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

62:   uint public constant nearbyRanges = 2;

```

```solidity
File: /contracts/RangeManager.sol

36:   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);

```

```solidity
File: /contracts/TokenisableRange.sol

58:   INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);

59:   IUniswapV3Factory constant public V3_FACTORY = IUniswapV3Factory(0x1F98431c8aD98523631AE4a59f267346ea31F984);

60:   address constant public treasury = 0x22Cc3f665ba4C898226353B672c5123c58751692;

61:   uint constant public treasuryFee = 20;

```

### [GAS-14] Use shift Right/Left instead of division/multiplication if possible

#### Description:

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

353:       depositAndStash(ticks[tick0Index], availToken0 / 2, 0);

354:       depositAndStash(ticks[tick0Index+1], availToken0 / 2, 0);

357:       depositAndStash(ticks[tick1Index], 0, availToken1 / 2);

358:       depositAndStash(ticks[tick1Index+1], 0, availToken1 / 2);

457:     if (adjustedBaseFeeX4 > baseFeeX4 * 3 / 2) adjustedBaseFeeX4 = baseFeeX4 * 3 / 2;

```

```solidity
File: /contracts/TokenisableRange.sol

67:       uint z = (x + 1) / 2;

105:       midleTick = (_upperTick + _lowerTick) / 2;

```

```solidity
File: /contracts/helper/LPOracle.sol

45:     uint z = (x + 1) / 2;

```

### [GAS-15] Splitting require() statements that use && saves gas

#### Description:

Instead of using operator `&&` on a single `require`. Using a two `require` can save more gas.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

120:     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

141:     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

170:     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

```

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

167:     require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");

495:     require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );

523:     require ( priceAssetA > 0 && priceAssetB > 0, "OPM: Invalid Oracle Price");

536:     require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");

```

```solidity
File: /contracts/RangeManager.sol

108:     require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");

```

```solidity
File: /contracts/TokenisableRange.sol

209:       require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

271:       require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");

```

```solidity
File: /contracts/helper/LPOracle.sol

29:     require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");

29:     require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");

101:     require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");

```

### [GAS-16] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
Note that this optimization seems to be dependent on usage of a more recent Solidity version. The following gas savings are on version 0.8.17.
[Code4arena example](https://code4rena.com/reports/2022-10-traderjoe/#g-03-ternary-operation-is-cheaper-than-if-else-statement)

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

274: if (tSupply == 0 || vaultValueX8 == 0)
      liquidity = valueX8 * 1e10;
    else {
      liquidity = tSupply * valueX8 / vaultValueX8;
    }

```

```solidity
File: /contracts/helper/LPOracle.sol

89: if (decimalsB >= decimalsA) {
      norm_b = sqrt( a * b * priceA * 10**(decimalsB-decimalsA) / priceB );
    } else {
      norm_b = sqrt( a * b * priceA / 10**(decimalsA-decimalsB) / priceB );
    }

```

### [GAS-17] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol

45:     uint8 mode = abi.decode(params, (uint8) );

45:     uint8 mode = abi.decode(params, (uint8) );

48:       (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));

53:       (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));

71:     for ( uint8 k = 0; k<assets.length; k++){

93:     for ( uint8 k =0; k<assets.length; k++){

172:     for (uint8 i = 0; i< options.length; ){

203:     for (uint8 i = 0; i< options.length; ){

339:     (ERC20 token0, uint8 decimals0) = TokenisableRange(debtAsset).TOKEN0();

340:     (ERC20 token1, uint8 decimals1) = TokenisableRange(debtAsset).TOKEN1();

```

```solidity
File: /contracts/TokenisableRange.sol

28:   int24 public lowerTick;

29:   int24 public upperTick;

30:   uint24 public feeTier;

38:     uint8 decimals;

99:     int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );

100:     int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );

104:       int24 midleTick;

106:       _upperTick = (midleTick + int24(feeTier)) - (midleTick + int24(feeTier)) % int24(feeTier * 2);

106:       _upperTick = (midleTick + int24(feeTier)) - (midleTick + int24(feeTier)) % int24(feeTier * 2);

106:       _upperTick = (midleTick + int24(feeTier)) - (midleTick + int24(feeTier)) % int24(feeTier * 2);

107:       _lowerTick = _upperTick - int24(feeTier) - int24(feeTier);

107:       _lowerTick = _upperTick - int24(feeTier) - int24(feeTier);

112:       _lowerTick = (_lowerTick + int24(feeTier)) - (_lowerTick + int24(feeTier)) % int24(feeTier * 2);

112:       _lowerTick = (_lowerTick + int24(feeTier)) - (_lowerTick + int24(feeTier)) % int24(feeTier * 2);

112:       _lowerTick = (_lowerTick + int24(feeTier)) - (_lowerTick + int24(feeTier)) % int24(feeTier * 2);

113:       _upperTick = (_upperTick + int24(feeTier)) - (_upperTick + int24(feeTier)) % int24(feeTier * 2);

113:       _upperTick = (_upperTick + int24(feeTier)) - (_upperTick + int24(feeTier)) % int24(feeTier * 2);

113:       _upperTick = (_upperTick + int24(feeTier)) - (_upperTick + int24(feeTier)) % int24(feeTier * 2);

```

```solidity
File: /contracts/helper/FixedOracle.sol

33:             uint80 roundId,

37:             uint80 answeredInRound

40:         roundId = uint80(block.number);

```

```solidity
File: /contracts/helper/LPOracle.sol

8:   function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);

14:   function decimals() external view returns (uint8);

22: 	uint8 public immutable decimalsA;

23: 	uint8 public immutable decimalsB;

38:   function decimals() external pure returns (uint8) {

```

```solidity
File: /contracts/helper/V3Proxy.sol

13:         uint24 fee;

25:         uint24 fee;

42:         uint24 fee,

50:         uint24 fee,

64:     uint24      immutable public feeTier;

75:     constructor(ISwapRouter _router, IQuoter _quoter, uint24 _fee) {

```

### [GAS-18] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

72:     string memory name,

73:     string memory symbol,

```

```solidity
File: /contracts/RangeManager.sol

75:   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

75:   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

```

```solidity
File: /contracts/TokenisableRange.sol

45:   string _name;

46:   string _symbol;

85:   function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {

85:   function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {

96:     string memory quoteSymbol = asset0.symbol();

97:     string memory baseSymbol  = asset1.symbol();

108:       _name     = string(abi.encodePacked("Ticker ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));

109:      _symbol    = string(abi.encodePacked("T-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));

114:       _name     = string(abi.encodePacked("Ranger ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));

115:       _symbol   = string(abi.encodePacked("R-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));

125:   function name()     public view virtual override returns (string memory) { return _name; }

127:   function symbol()   public view virtual override returns (string memory) { return _symbol; }

```

### [GAS-19] Can make the variable outside the loop to save gas

#### Description:

Make it outside and only use it inside.

https://github.com/crytic/slither/wiki/Detector-Documentation#costly-operations-inside-a-loop

#### **Proof Of Concept**

```solidity
File: /contracts/GeVault.sol

299:     for (uint k = 0; k < ticks.length; k++){

314:     for (uint k = 0; k < ticks.length; k++){

393:     for(uint k=0; k<ticks.length; k++){

```
