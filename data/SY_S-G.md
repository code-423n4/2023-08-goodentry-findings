## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] |Splitting require() statements taht use && saves gas | |7| | 
| [G-02] |Don't apply the same value to state variables | |1| | 
| [G-03] |Using calldata instead of memory for read-only arguments in external functions saves gas | |3| | 
| [G-04] |Unused named return variables without optimizer waste gas | |4| | 
| [G-05] |Duplicated require()/if() checks should be refactored to a modifier or function | |5| | 
| [G-06] |Require()/Revert() string longer than 32 bytes cost extra gas | |1| |  
| [G-07] |Use assembly to write address storage values | |2| | 
| [G-08] |State varaibles only set in the Constructor should be declared Immutable | |15| | 
| [G-09] |Using PRIVATE rather than PUBLIC FOR Constants, Saves Gas  | |6| | 
| [G-10] |Can make the variable outside the loop to save gas  | |5| | 
| [G-11] |Functions guaranteed to when called by normal users can be marked Payable | |14| | 
| [G-12] |Use assembly for math (add, sub, mul, div)  | |9| | 
| [G-13] |Should use arguments instead of state variable | |1| | 
| [G-14] |Avoid contract existence checks by using low level calls | |6| | 
| [G-15] |Sort Solidity operations using short-circuit mode  | |2| | 
| [G-16] | Not using the named return variable when a function returns, wastes deployment gas | |4| | 
| [G-17] |Before transfer of  some functions, we should check some variables for possible gas save | |1| | 
| [G-18] |Variable names that consist of all capital letters should be reserved for constant/immutable variables | |11| | 
| [G-19] |Empty blocks should be removed or emit something | |2| | 
| [G-20] |2**<N> should be re-written as type(uint<N>).max | |3| | 
| [G-21] |Use constants instead of type(uintx).max | |3| | 
| [G-22] |abi.encode() is less efficient than  abi.encodepacked() | |2| | 
| [G-23] |Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)  | |1| | 
| [G-24] |Require() or revert() statements that check input arguments should be at the top of the function | |6| | 
| [G-25] |Use hardcode address instead address(this) | |5| | 
| [G-26] |Using delete statement can save gas | |1| | 





## Gas Optimizations  

## [G-1] Splitting require() statements taht use && saves gas  

Instead of using operator && on a single require. Using a two require can save more gas.
i.e. for require(version == 1 && bytecodeHash[1] == bytes1(0) , “zf”)	; use:
require(version == 1); require(_bytecodeHash[1] == bytes(0));

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

## [G-2] Don't apply the same value to state variables

Assigning the same value to a state variable repeatedly can cause unnecessary gas costs, because each assignment operation incurs a gas cost. Additionally, if the value being assigned is the same as the current value of the state variable, the assignment operation does not actually change anything, which can waste gas.

```solidity
file: /contracts/GeVault.sol

84    token0 = ERC20(_token0);
85    token1 = ERC20(_token1);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L84-L85
## [G-3] Using calldata instead of memory for read-only arguments in external functions saves gas 

calldata must be used when declaring  an external function's dynamic parameters.
When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

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
## [G-4] Unused named return variables without optimizer waste gas

Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable

```solidity
file: /contracts/TokenisableRange.sol

352     function getValuePerLPAtPrice(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 priceX1e8) {

361     function latestAnswer() public view returns (uint256 priceX1e8) {    

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352


```solidity
file: /contracts/PositionManager/PositionManager.sol

52     ) virtual external returns (bool result) {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L52


```solidity
file: /contracts/helper/OracleConvert.sol
 
 78       function latestRoundData() external view returns (uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound) {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/#L78
## [G-5]  Duplicated require()/if() checks should be refactored to a modifier or function   

 to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.


```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol



69        require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");



393    require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L69


```solidity
file: /contracts/GeVault.sol


120     require(t0 == token0 && t1 == token1, "GEV: Invalid TR");




203    require(poolMatchesOracle(), "GEV: Oracle Error");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120


```solidity
file: /contracts/GeVault.sol


121     if (ticks.length == 0) ticks.push(t);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L121
## [G-6] Require()/Revert() string longer than 32 bytes cost extra gas  

when these strings get longer than 32 bytes, they cost more gas
Use shorter require()/revert() strings

```solidity
file: /contracts/helper/FixedOracle.sol

.

11        require(msg.sender == owner, "Only the owner can call this function.");

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11
## [G-7] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```solidity
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```


```solidity
file: /contracts/GeVault.sol

109    treasury = newTreasury; 

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L109


```solidity
file: /contracts/RoeRouter.sol

85    treasury = newTreasury;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L85
## [G-8] State varaibles only set in the Constructor should be declared Immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

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
file: /contracts/RangeManager.sol

50    LENDING_POOL = lendingPool;

51    ASSET_0 = _asset0 < _asset1 ? _asset0 : _asset1;

52    ASSET_1 = _asset0 < _asset1 ? _asset1 : _asset0;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L50-L52


```solidity
file: /contracts/PositionManager/PositionManager.sol

31    ROEROUTER = RoeRouter(roerouter);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L31


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
## [G-9] Using PRIVATE rather than PUBLIC FOR Constants, Saves Gas        

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
## [G-10] Can make the variable outside the loop to save gas

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
## [G-11] Functions guaranteed to when called by normal users can be marked Payable

The onlyOwner modifier makes a function revert if not called by the address registered as the owner

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


```solidity
file: /contracts/helper/V3Proxy.sol

94     function emergencyWithdraw(ERC20 token) onlyOwner external {   

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L94


```solidity
file: /contracts/RangeManager.sol

75     function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

95     function initRange(address tr, uint amount0, uint amount1) external onlyOwner {
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75


```solidity
file: /contracts/RoeRouter.sol

48    function deprecatePool(uint poolId) public onlyOwner {

65    public onlyOwner 

83    function setTreasury(address newTreasury) public onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L48


```solidity
file: /contracts/helper/FixedOracle.sol
 
24    function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24
## [G-12] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.


```solidity
file: /contracts/TokenisableRange.sol

67      uint z = (x + 1) / 2;

71          z = (x / z + z) / 2;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L67


```solidity
file: /contracts/helper/LPOracle.sol

94    uint norm_a = a * b / norm_b;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L94


```solidity
file: /contracts/GeVault.sol

374    priceX8 = priceX8 * uint(sqrtPriceX96 / 2 ** 12) ** 2 * 1e8 / 2**168;

451      adjustedBaseFeeX4 = baseFeeX4 * value0 / (value1 + 1);

456    if (adjustedBaseFeeX4 < baseFeeX4 / 2) adjustedBaseFeeX4 = baseFeeX4 / 2;

457    if (adjustedBaseFeeX4 > baseFeeX4 * 3 / 2) adjustedBaseFeeX4 = baseFeeX4 * 3 / 2;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L374


```solidity
file: /contracts/TokenisableRange.sol

183    fee0 = fee0 + newFee0 - tf0;

184    fee1 = fee1 + newFee1 - tf1;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L183-L184
## [G-13] Should use arguments instead of state variable 

state variables should not used in emit  ,  This will save near 97 gas  

```solidity
file: /contracts/GeVault.sol

362    emit Rebalance(tickIndex);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362
## [G-14] Avoid contract existence checks by using low level calls		

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
## [G-15] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.


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
## [G-16] Not using the named return variable when a function returns, wastes deployment gas

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
## [G-17] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /contracts/RangeManager.sol
 
96    ASSET_0.safeTransferFrom(msg.sender, address(this), amount0);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L96
## [G-18]  Variable names that consist of all capital letters should be reserved for constant/immutable variables

If the variable needs to be different based on which class it comes from, a view/pure function should be used instead 

```solidity
file: /contracts/TokenisableRange.sol

41    ASSET public TOKEN0;

42    ASSET public TOKEN1;

43    IAaveOracle public ORACLE;

54  address public TREASURY_DEPRECATED = 0x22Cc3f665ba4C898226353B672c5123c58751692;

55  uint public treasuryFee_deprecated = 20;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L41-L43


```solidity
file: /contracts/RangeManager.sol

27     ILendingPool public LENDING_POOL;

32     ERC20 public ASSET_0;

33     ERC20 public ASSET_1;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L27


```solidity
file: /contracts/PositionManager/PositionManager.sol

21     ILendingPoolAddressesProvider public ADDRESSES_PROVIDER; // IFlashLoanReceiver  requirement

22     ILendingPool public LENDING_POOL; // IFlashLoanReceiver  requirement

23     RoeRouter public ROEROUTER; 

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L21-L23
## [G-19] Empty blocks should be removed or emit something

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
## [G-20]  2**<N> should be re-written as type(uint<N>).max

Earlier versions of solidity can use uint<n>(-1) instead. Expressions not including the - 1 can often be re-written to accomodate the change (e.g. by using a > rather than a >=, which will also save some gas)

```solidity
file: /contracts/GeVault.sol

374    priceX8 = priceX8 * uint(sqrtPriceX96 / 2 ** 12) ** 2 * 1e8 / 2**168;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L374


```solidity
file: /contracts/TokenisableRange.sol

99      int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) );


100      int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) );

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L99-L100
## [G-21]  Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /contracts/TokenisableRange.sol

172        amount0Max: type(uint128).max,

173        amount1Max: type(uint128).max

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L172-L173


```solidity
file: /contracts/helper/OracleConvert.sol

79    return (type(uint80).max, latestAnswer(), block.timestamp, block.timestamp, type(uint80).max);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/#L79
## [G-22] abi.encode() is less efficient than  abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

168    bytes memory params = abi.encode(0, poolId, msg.sender, sourceSwap);

199    bytes memory params = abi.encode(1, poolId, user, collateralAsset); // mode = 1 -> liquidation

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L168
## [G-23] Caching global variables is more expensive than using the actual variable (use msg.sender instead of caching it)

```solidity
file: /contracts/TokenisableRange.sol

88    creator = msg.sender;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L88
## [G-24] Require() or revert() statements that check input arguments should be at the top of the function  

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

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
## [G-25] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address


 if the contract's address is needed in the code, it's more gas-efficient to hardcode the address as a constant rather than using the address(this) expression. This is because using address(this) requires additional gas consumption to compute the contract's address at runtime.

Here's an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```


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
file: /contracts/helper/V3Proxy.sol

167        amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountOut, amountInMax, 0));         

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L167


```solidity
file: /contracts/RangeManager.sol

118        trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L118
## [G-26] Using delete statement can save gas

```solidity
file: /contracts/PositionManager/OptionsPositionManager.sol

204      flashtype[i] = 0; // dont open debt for liquidations, need to repay

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L204



