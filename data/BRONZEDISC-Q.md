## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol

```solidity
// place this even after all state variables
4:    event SetHardcodedPrice(int256 price);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol

```solidity
// place these events after state variable
28:  event Withdraw(address user, address asset, uint amount);
29:  event Deposit(address user, address asset, uint amount);
30:  event AddRange(uint128 startX10, uint128 endX10, uint step);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol

```solidity
// place this struct before all the functions in this interface
22:    struct ExactOutputSingleParams {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol

```solidity
// place these events after state variables
21:  event InitTR(address asset0, address asset1, uint128 startX10, uint128 endX10);
22:  event Deposit(address sender, uint trAmount);
23:  event Withdraw(address sender, uint trAmount);
24:  event ClaimFees(uint fee0, uint fee1);
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

```solidity
// place this receive function right after the constructor
462:  receive() external payable {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

```solidity
// place these events after all the state variables.
30:  event Deposit(address indexed sender, address indexed token, uint amount, uint liquidity);
31:  event Withdraw(address indexed sender, address indexed token, uint amount, uint liquidity);
32:  event PushTick(address indexed ticker);
33:  event ShiftTick(address indexed ticker);
34:  event ModifyTick(address indexed ticker, uint index);
35:  event Rebalance(uint tickIndex);
36:  event SetEnabled(bool isEnabled);
37:  event SetTreasury(address treasury);
38:  event SetFee(uint baseFeeX4);
39:  event SetTvlCap(uint tvlCap);
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/

```solidity
// place this internal function after all the other ones.
37:  function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {

// place this external function before all the other ones.
78:  function latestRoundData() external view returns (uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol

```solidity
// place these internal functions for last
44:  function sqrt(uint x) internal pure returns (uint y) {
56:  function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol

```solidity
// place these internal functions after the external one.
63:  function getPoolAddresses(uint poolId) 
80:  function checkSetAllowance(address token, address spender, uint amount) internal {
89:  function cleanup(ILendingPool LP, address user, address asset) internal {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol

```solidity
// place these internal functions for last since there are no private ones.
59:  function checkNewRange(uint128 start, uint128 end) internal view {
107:  function removeFromStep(uint256 step) internal {
151:  function transferAssetsIntoStep(TokenisableRange tr, uint256 step, uint256 amount0, uint256 amount1) internal {
190:  function cleanup() internal {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol

```solidity
// place these internal ones for last since there is no private ones.
66:  function sqrt(uint x) internal pure returns (uint y) {
333:  function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {

// place these external functions before all the other ones
85:  function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) 134:  function init(uint n0, uint n1) external {
222:  function deposit(uint256 n0, uint256 n1) external nonReentrant returns (uint256 lpAmt) {
292:  function withdraw(uint256 lp, uint256 amount0Min, uint256 amount1Min) external nonReentrant returns (uint256 removed0, uint256 removed1) {
377:  function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

```solidity
// place this external function on top right after the receive function
289:  function latestAnswer() external view returns (uint256 priceX8) {

// internal functions coming before public ones
313:  function removeFromAllTicks() internal {
321:  function removeFromTick(uint index) internal {
337:  function deployAssets() internal {
404:  function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity){

// place private functions for last
385:  function checkSetApprove(address token, address spender, uint amount) private {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol

```solidity
// internal function coming before external ones
61:  function executeBuyOptions(
83:  function executeLiquidation(
250:  function closeDebt(
336:  function checkExpectedBalances(address debtAsset, uint debtAmount, uint token0Amount, uint token1Amount) internal view
357:  function calculateAndSendFee(

// private function should come for last
123:  function withdrawOptionAssets(

// public functions coming before external ones
439:  function swapTokens(uint poolId, address sourceAsset, uint amount) public returns (uint received) {
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/

```solidity
// @return missing
30:    function decimals() external pure returns (uint8) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol

```solidity
3:  contract HardcodedPriceOracle {
4:    event SetHardcodedPrice(int256 price);
10:    modifier onlyOwner() {
15:    constructor(int256 _hardcodedPrice) {
20:    function latestAnswer() external view returns (int256) {
24:    function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {
29:    function latestRoundData()
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol

```solidity
12:  event AddPool(uint poolId, address lendingPoolAddressProvider);
13:  event DeprecatePool(uint poolId);
14:  event UpdateTreasury(address treasury);
23:  struct RoePool {
40:  function getPoolsLength() public view returns (uint poolLength) {

// @return missing
59:  function addPool(
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol

```solidity
6:  interface UniswapV2Pair {
7:  function totalSupply() external view returns (uint);
8:  function getReserves() external view returns (uint112 reserve0, uint112 reserve1, uint32 blockTimestampLast);
9:  function token0() external view returns (address);
10:  function token1() external view returns (address);
13:  interface IERC20 {
14:  function decimals() external view returns (uint8);
17:  contract LPOracle {

// @return missing
38:  function decimals() external pure returns (uint8) {
44:  function sqrt(uint x) internal pure returns (uint y) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol

```solidity
17:  contract PositionManager is IFlashLoanReceiver {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol

```solidity
48:  constructor(ILendingPool lendingPool, ERC20 _asset0, ERC20 _asset1)  {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol

```solidity
9:  interface ISwapRouter {
10:    struct ExactInputSingleParams {
20:    function exactInputSingle(ExactInputSingleParams calldata params) external payable returns (uint256 amountOut);
22:    struct ExactOutputSingleParams {
32:    function exactOutputSingle(ExactOutputSingleParams calldata params) external payable returns (uint256 amountIn);
33:    function refundETH() external payable;
34:    function WETH9() external view returns (address);
38:  interface IQuoter {
39:    function quoteExactInputSingle(
47:    function quoteExactOutputSingle(
56:  interface IWETH9 is IERC20 {
57:    function withdraw(uint) external;
60:  contract V3Proxy is ReentrancyGuard, Ownable {
67:    event Swap(
75:    constructor(ISwapRouter _router, IQuoter _quoter, uint24 _fee) {
82:    function WETH() external view returns (address) {
86:    receive() external payable {
90:    fallback() external payable {
94:    function emergencyWithdraw(ERC20 token) onlyOwner external {   
98:    function getAmountsOut(uint amountIn, address[] calldata path) external returns (uint[] memory amounts) {
105:    function getAmountsIn(uint amountOut, address[] calldata path) external returns (uint[] memory amounts) {
112:    function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {
124:    function swapTokensForExactTokens(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {
137:    function swapExactETHForTokens(uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
147:    function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
160:    function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
178:    function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol

```solidity
18:  contract TokenisableRange is ERC20("", ""), ReentrancyGuard {
21:  event InitTR(address asset0, address asset1, uint128 startX10, uint128 endX10);
22:  event Deposit(address sender, uint trAmount);
23:  event Withdraw(address sender, uint trAmount);
24:  event ClaimFees(uint fee0, uint fee1);
343:  function returnExpectedBalance(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 amt0, uint256 amt1) {


// @return missing
125:  function name()     public view virtual override returns (string memory) { return _name; }
127:  function symbol()   public view virtual override returns (string memory) { return _symbol; }
292:  function withdraw(uint256 lp, uint256 amount0Min, uint256 amount1Min) external nonReentrant returns (uint256 removed0, uint256 removed1) {
333:  function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {
352:  function getValuePerLPAtPrice(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 priceX1e8) {
361:  function latestAnswer() public view returns (uint256 priceX1e8) {
368:  function getTokenAmountsExcludingFees(uint amount) public view returns (uint token0Amount, uint token1Amount){
377:  function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

```solidity
30:  event Deposit(address indexed sender, address indexed token, uint amount, uint liquidity);
31:  event Withdraw(address indexed sender, address indexed token, uint amount, uint liquidity);
32:  event PushTick(address indexed ticker);
33:  event ShiftTick(address indexed ticker);
34:  event ModifyTick(address indexed ticker, uint index);
35:  event Rebalance(uint tickIndex);
36:  event SetEnabled(bool isEnabled);
37:  event SetTreasury(address treasury);
38:  event SetFee(uint baseFeeX4);
39:  event SetTvlCap(uint tvlCap);
67:  constructor(
337:  function deployAssets() internal { 

// @return missing
247:  function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity) 
298:  function getReserves() public view returns (uint amount0, uint amount1){
367:  function poolMatchesOracle() public view returns (bool matches){
428:  function getActiveTickIndex() public view returns (uint activeTickIndex) {

// @param missing
321:  function removeFromTick(uint index) internal {
404:  function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity){
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol

```solidity
7:  interface  AmountsRouter {
8:  function getAmountsOut(uint256 amountIn, address[] calldata path) external returns (uint256[] memory amounts);
9:  function getAmountsIn(uint256 amountOut, address[] calldata path) external returns (uint256[] memory amounts);

12:  contract OptionsPositionManager is PositionManager {
16:  event BuyOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);
17:  event SellOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);
18:  event ClosePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);
19:  event LiquidatePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);
20:  event ReducedPosition(address indexed user, address indexed asset, uint amount);
21:  event Swap(address indexed user, address sourceAsset, uint sourceAmount, address targetAsset, uint targetAmount);
61:  function executeBuyOptions(
83:  function executeLiquidation(

// @return missing
123:  function withdrawOptionAssets(
250:  function closeDebt(
357:  function calculateAndSendFee(
544:  function addDust(address debtAsset, address token0, address token1) internal returns (uint amount){
```
---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/

```solidity
// private and internal `functions` should preppend with `underline`
37:  function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol

```solidity
6:    int256 private hardcodedPrice;
7:    address private owner;
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol

```solidity
44:  function sqrt(uint x) internal pure returns (uint y) {
56:  function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol

```solidity
//  private and internal `functions` should preppend with `underline`
63:  function getPoolAddresses(uint poolId) 
80:  function checkSetAllowance(address token, address spender, uint amount) internal {
89:  function cleanup(ILendingPool LP, address user, address asset) internal {
124:  function PMWithdraw(ILendingPool LP, address user, address asset, uint amount) internal {
138:  function validateValuesAgainstOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB, uint amountB) internal view {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol

```solidity
//  private and internal `functions` should preppend with `underline`
59:  function checkNewRange(uint128 start, uint128 end) internal view {
107:  function removeFromStep(uint256 step) internal {
151:  function transferAssetsIntoStep(TokenisableRange tr, uint256 step, uint256 amount0, uint256 amount1) internal {
190:  function cleanup() internal {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol

```solidity
// private and internal `variables` should preppend with `underline`
50:  address private creator;

// private and internal `functions` should preppend with `underline`
66:  function sqrt(uint x) internal pure returns (uint y) {
333:  function returnExpectedBalanceWithoutFees(uint TOKEN0_PRICE, uint TOKEN1_PRICE) internal view returns (uint256 amt0, uint256 amt1) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

```solidity
//  private and internal `functions` should preppend with `underline`
313:  function removeFromAllTicks() internal {
321:  function removeFromTick(uint index) internal {
337:  function deployAssets() internal { 
404:  function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity){
385:  function checkSetApprove(address token, address spender, uint amount) private {
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol

```solidity
// private and internal `functions` should preppend with `underline`
61:  function executeBuyOptions(
83:  function executeLiquidation(
250:  function closeDebt(
336:  function checkExpectedBalances(address debtAsset, uint debtAmount, uint token0Amount, uint token1Amount) internal view
357:  function calculateAndSendFee(
470:  function swapExactTokensForTokens(IUniswapV2Router01 ammRouter, IPriceOracle oracle, uint amount, address[] memory path) 
491:  function swapTokensForExactTokens(IUniswapV2Router01 ammRouter, uint recvAmount, uint maxAmount, address[] memory path) internal {
513:  function getTargetAmountFromOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB) 
533:  function sanityCheckUnderlying(address tr, address token0, address token1) internal {
544:  function addDust(address debtAsset, address token0, address token1) internal returns (uint amount){
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol

```solidity
// lock this to a single version by removing the ^.
1:  pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol

```solidity
// lock this to a single version by removing the ^.
2:  pragma solidity ^0.8.0;
```
