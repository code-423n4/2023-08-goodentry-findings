|     |
| --- |
| G‑01\] Using calldata instead of memory for read-only arguments in external functions saves gas |
| \[G-02\] abi.encode() is less efficient than abi.encodePacked() |
| \[G-03\] Make for loop unchecked |
| \[G-04\] If statements that use && can be refactored into nested if statements |
| \[G-05\] Avoiding Initialization of the variables to it's default value. |
| \[G-06\] Splitting require() statements that use && saves gas - (saves 8 gas per &&) |
| \[G-07\] Functions guaranteed to revert when called by normal users can be marked payable |
| \[G-08\] Sort Solidity operations using short-circuit mode |
| \[G-09\] Rearranging order of state variable declarations to pack them will save storage slots and gas in the following way. |
| \[G‑10\] Using private rather than public for constants, saves gas |
| \[G-11\]An implementation of a square root function was added to the [dapp-bin](https://github.com/ethereum/dapp-bin) as part of a maths library a while back. |
| \[G-12\] Massive 15k per tx gas savings - use 1 and 2 for Reentrancy guard |
| \|\[G-11\]\|Use assembly for math (add, sub, mul, div) |
| \[G-12\]\|internal functions not called by the contract should be removed to save deployment gas |

## \*\*\*

## \[G‑01\] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. Each iteration of this for-loop costs at least 60 gas (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having `memory` arguments, it’s still valid for implementation contracs to use `calldata` arguments instead.

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it’s still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

```Solidity
  function buyOptions(
    uint poolId, 
    address[] memory options, 
    uint[] memory amounts, 
    address[] memory sourceSwap
  )
    external
  {
```

```Solidity
 function liquidate (
    uint poolId, 
    address user,
    address[] memory options, 
    uint[] memory amounts,
    address collateralAsset
  )
    external
  {
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L189C2-L197C4

## \[G-02\] abi.encode() is less efficient than abi.encodePacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost
`bytes memory params = abi.encode(0, poolId, msg.sender, sourceSwap);`
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L168

`bytes memory params = abi.encode(1, poolId, user, collateralAsset);`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L199

## \[G-03\] Make for loop unchecked

The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time and gas to let the for loop overflow.
`for ( uint8 k = 0; k<assets.length; k++){`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L71

`for ( uint8 k =0; k<assets.length; k++){`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L93

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L314C7-L314C7

## \[G-04\] If statements that use && can be refactored into nested if statements

`if ( repayAmount > 0 && repayAmount < debt ) debt = repayAmount;`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L234

`if (oraclePrice < priceX8 * 101 / 100 && oraclePrice > priceX8 * 99 / 100) matches = true;`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L377

`if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L434

`if ((fee0 * 100 > bal0 ) && (fee1 * 100 > bal1)) {`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L190

```
    if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) ){
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L237

`if ( newFee0 == 0 && newFee1 == 0 ){`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L268

## \[G-05\] Avoiding Initialization of the variables to it's default value.

`for ( uint8 k = 0; k<assets.length; k++){`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L71

`for ( uint8 k =0; k<assets.length; k++){`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L93

`for (uint8 i = 0; i< options.length; ){`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L172

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L299

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L314C7-L314C7

## \[G-06\] Splitting require() statements that use && saves gas - (saves 8 gas per &&)

```
167: require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L167

```
 require( 
      (debtValue < 1e8 && tokensValue < 1e8 )
      || (tokensValue > debtValue * 98 / 100 && tokensValue < debtValue * 102 / 100), 
      "OPM: Slippage Error"
    );
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L344C4-L348C7

```
495:     require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L495

```
require ( priceAssetA > 0 && priceAssetB > 0, "OPM: Invalid Oracle Price");
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L523

```
require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L536

`require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L271

```
 require(clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/OracleConvert.sol#L23

## \[G-07\] Functions guaranteed to revert when called by normal users can be marked payable

> If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

```
function setEnabled(bool _isEnabled) public onlyOwner { 
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L101

```
function setTreasury(address newTreasury) public onlyOwner { 
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L108

```
 function pushTick(address tr) public onlyOwner {
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L116

```
function shiftTick(address tr) public onlyOwner {
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L137

```
 function modifyTick(address tr, uint index) public onlyOwner {
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L167

```
  function setBaseFee(uint newBaseFeeX4) public onlyOwner {
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L183

```
  function setTvlCap(uint newTvlCap) public onlyOwner {
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L191

### Recommended Mitigation Steps

> Functions guaranteed to revert when called by normal users can be marked payable.

## \[G-08\] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
    //Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

```
 if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L434

```
    if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) ){
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L237

## \[G-09\] Rearranging order of state variable declarations to pack them will save storage slots and gas in the following way.

```
file : contracts/TokenisableRange.sol

  uint128 public liquidity;
  int24 public lowerTick;
  int24 public upperTick;
  uint24 public feeTier;
  
  uint256 public tokenId;
  uint256 public fee0;
  uint256 public fee1;
  
  // @notice deprecated, keep to avoid beacon storage slot overwriting errors
  address public TREASURY_DEPRECATED = 0x22Cc3f665ba4C898226353B672c5123c58751692;
  uint public treasuryFee_deprecated = 20;
  

```

## \[G‑10\] Using private rather than public for constants, saves gas

`uint constant public treasuryFee = 20;`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L61

## \[G-11\]An implementation of a square root function was added to the [dapp-bin](https://github.com/ethereum/dapp-bin) as part of a maths library a while back.

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L66C1-L73C4

**you can simply use it**

## \[G-12\] Massive 15k per tx gas savings - use 1 and 2 for Reentrancy guard

Using true and false will trigger gas-refunds, which after London are 1/5 of what they used to be, meaning using 1 and 2 (keeping the slot non-zero), will cost 5k per change (5k + 5k) vs 20k + 5k, saving you 15k gas per function which uses the modifier.

```
 function deposit(uint256 n0, uint256 n1) external nonReentrant returns (uint256 lpAmt) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L222

```
  function withdraw(uint256 lp, uint256 amount0Min, uint256 amount1Min) external nonReentrant returns (uint256 removed0, uint256 removed1) {
```

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L292

## \[G‑13\] Use a more recent version of Solidity

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

`pragma solidity ^0.8.0;`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L2

`pragma solidity ^0.8.0;`
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/FixedOracle.sol#L1