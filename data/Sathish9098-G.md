##

## [G-1] Using ``calldata`` instead of memory for read-only arguments in external functions saves gas

When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Note that even if an interface defines a function as having memory arguments, it’s still valid for implementation contracs to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it’s still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract, and cases where a constructor is involved

### ``options, amounts, sourceSwap`` can be changed to calldata instead of memory. Because read-only arguments : Saves ``846 GAS``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L159-L165

```diff
FILE: 2023-08-goodentry/contracts/PositionManager/OptionsPositionManager.sol

159: function buyOptions(
160:    uint poolId, 
+ 161:    address[] calldata options, 
- 161:    address[] memory options, 
+ 162:    uint[] calldata amounts, 
- 162:    uint[] memory amounts, 
+ 163:    address[] memory sourceSwap
- 163:    address[] memory sourceSwap
164:  )
165:    external
166:  {

```

### ``options, amounts`` can be changed to ``calldata`` instead of ``memory``. Because read-only arguments: Saves ``564 GAS``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L189-L196

```diff
FILE: 2023-08-goodentry/contracts/PositionManager/OptionsPositionManager.sol

189: function liquidate (
190:    uint poolId, 
191:    address user,
+ 192:    address[] memory options, 
- 192:    address[] memory options, 
+ 193:    uint[] memory amounts,
- 193:    uint[] memory amounts,
194:    address collateralAsset
195:  )
196:    external
197:  {

```

### ``startName, endName`` can be changed to ``calldata`` instead of ``memory``. Because read-only arguments : Saves ``564 GAS``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L75


```diff
FILE: 2023-08-goodentry/contracts/RangeManager.sol

- 75: function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

+ 75: function generateRange(uint128 startX10, uint128 endX10, string calldata startName, string calldata endName, address beacon) external onlyOwner {

```
##

## [G-2] State variables can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

### ``baseFeeX4`` can be declared uint96 instead of ``uint`` : Saves ``2000 GAS``, ``1 SLOT``

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L48-L54

``baseFeeX4`` is not exceed ``1e4`` this because ``newBaseFeeX4 < 1e4`` check

```diff
FILE: Breadcrumbs2023-08-goodentry/contracts/GeVault.sol

48:  ERC20 public token0;
+ 51:  /// @notice Pool base fee 
+ 52:  uint96 public baseFeeX4 = 20;
49:  ERC20 public token1;
50:  bool public isEnabled = true;
- 51:  /// @notice Pool base fee 
- 52:  uint public baseFeeX4 = 20;
53:  /// @notice Max vault TVL with 8 decimals
54:  uint public tvlCap = 1e12;

```

Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata

Condition checks should be top of the function. The cheep condition checks first then expensive checks 

don't emit state variable

combine both events together 

Using bools for storage incurs overhead

```
    // Booleans are more expensive than uint256 or any type that takes up a full
    // word because each write operation emits an extra SLOAD to first read the
    // slot's contents, replace the bits taken up by the boolean, and then write
    // back. This is the compiler's defense against contract upgrades and
    // pointer aliasing, and it cannot be disabled.
```

Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from false to true, after having been true in the past

