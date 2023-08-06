
### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Inefficient use of abi.encode() | [Inefficient use of abi.encode()](#inefficient-use-of-abiencode) | 2 | 200 |
|[G-2] Use assembly to emit events | [Use assembly to emit events](#use-assembly-to-emit-events) | 35 | 1330 |
|[G-3] Assembly optimization for low level calls | [Assembly optimization for low level calls](#assembly-optimization-for-low-level-calls) | 3 | 744 |
|[G-4] Division operations between unsigned could be unchecked | [Division operations between unsigned could be unchecked](#division-operations-between-unsigned-could-be-unchecked) | 2 | 170 |
|[G-5] Modulus operations that could be unchecked | [Modulus operations that could be unchecked](#modulus-operations-that-could-be-unchecked) | 3 | 255 |
|[G-6] Constants Variable Should Be Private for Gas Optimization | [Constants Variable Should Be Private for Gas Optimization](#constants-variable-should-be-private-for-gas-optimization) | 6 | 20400 |
|[G-7] Use of emit inside a loop | [Use of emit inside a loop](#use-of-emit-inside-a-loop) | 3 | 3078 |
|[G-8] Identical Deployments Should be Replaced with Clone | [Identical Deployments Should be Replaced with Clone](#identical-deployments-should-be-replaced-with-clone) | 1 | - |
|[G-9] Greater or Equal Comparison Costs Less Gas than Greater Comparison | [Greater or Equal Comparison Costs Less Gas than Greater Comparison](#greater-or-equal-comparison-costs-less-gas-than-greater-comparison) | 5 | 15 |
|[G-10] Inefficient Parameter Storage | [Inefficient Parameter Storage](#inefficient-parameter-storage) | 2 | 100 |
|[G-11] Inefficient Gas Usage in Solidity Smart Contracts Due to Long Error Messages | [Inefficient Gas Usage in Solidity Smart Contracts Due to Long Error Messages](#inefficient-gas-usage-in-solidity-smart-contracts-due-to-long-error-messages) | 1 | 18 |
|[G-12] Optimal State Variable Order | [Optimal State Variable Order](#optimal-state-variable-order) | 1 | 5000 |
|[G-13] Short-circuit rules can be used to optimize some gas usage | [Short-circuit rules can be used to optimize some gas usage](#short-circuit-rules-can-be-used-to-optimize-some-gas-usage) | 2 | 4200 |
|[G-14] Safe Subtraction Should Be Unchecked | [Safe Subtraction Should Be Unchecked](#safe-subtraction-should-be-unchecked) | 7 | 595 |
|[G-15] Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 1 | - |
|[G-16] Redundant Contract Existence Check in Consecutive External Calls | [Redundant Contract Existence Check in Consecutive External Calls](#redundant-contract-existence-check-in-consecutive-external-calls) | 15 | 1500 |
|[G-17] Unused Named Return Variables | [Unused Named Return Variables](#unused-named-return-variables) | 3 | - |
|[G-18] Unused state variable | [Unused state variable](#unused-state-variable) | 1 | 2900 |
|[G-19] Usage of Custom Errors for Gas Efficiency | [Usage of Custom Errors for Gas Efficiency](#usage-of-custom-errors-for-gas-efficiency) | 80 | 22080 |

Total: 19 issues

#


## Inefficient Gas Usage in Solidity Smart Contracts Due to Long Error Messages
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 18

### Description
If the string parameter of a require() or revert() function is longer than 32 bytes, it can lead to inefficiencies. This is because each extra memory word of bytes past the original 32 incurs an MSTORE operation, which costs 3 gas.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/helper/FixedOracle.sol
```
 
Line: 11          require(msg.sender == owner, "Only the owner can call this function.")
```
 make the message shorter than 32 bytes

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11)


</details>

# 
 

## Inefficient use of abi.encode()
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 200

### Description
The `abi.encode()` function is less gas efficient than `abi.encodePacked()`, especially when encoding only static types. This detector identifies instances where `abi.encode()` is used and all arguments are static, suggesting that `abi.encodePacked()` could potentially be used instead for better gas efficiency. Note: `abi.encodePacked()` should not be used if any of the arguments are dynamic types, as it could lead to hash collisions.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: interfaces/PoolAddress.sol
```
 
Line: 35          pool = address(
          uint160(
              uint256(
                  keccak256(
                      abi.encodePacked(
                          hex'ff',
                          factory,
                          keccak256(abi.encode(key.token0, key.token1, key.fee)),
                          POOL_INIT_CODE_HASH
                      )
                  )
              )
          )
        )
```
 use  abi.encodepacked() instead.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L35-L48](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L35-L48)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 199          bytes memory params = abi.encode(1, poolId, user, collateralAsset)
```
 use  abi.encodepacked() instead.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L199](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L199)


</details>

# 


## Inefficient use of abi.encode()
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 200

### Description
The `abi.encode()` function is less gas efficient than `abi.encodePacked()`, especially when encoding only static types. This detector identifies instances where `abi.encode()` is used and all arguments are static, suggesting that `abi.encodePacked()` could potentially be used instead for better gas efficiency. Note: `abi.encodePacked()` should not be used if any of the arguments are dynamic types, as it could lead to hash collisions.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: interfaces/PoolAddress.sol
```
 
Line: 35          pool = address(
          uint160(
              uint256(
                  keccak256(
                      abi.encodePacked(
                          hex'ff',
                          factory,
                          keccak256(abi.encode(key.token0, key.token1, key.fee)),
                          POOL_INIT_CODE_HASH
                      )
                  )
              )
          )
        )
```
 use  abi.encodepacked() instead.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L35-L48](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L35-L48)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 199          bytes memory params = abi.encode(1, poolId, user, collateralAsset)
```
 use  abi.encodepacked() instead.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L199](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L199)


</details>

# 


## Use assembly to emit events
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 1330

### Description
This detector checks for instances where a contract uses assembly code to emit events. While it's technically possible to emit events using inline assembly in Solidity, it's generally discouraged due to readability concerns and potential for errors. Events should usually be emitted using higher-level Solidity constructs.

<details>

<summary>
There are 35 instances of this issue:

</summary>

###
- File: contracts/GeVault.sol
```
 
Line: 103          emit SetEnabled(_isEnabled)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L103](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L103)


- File: contracts/GeVault.sol
```
 
Line: 110          emit SetTreasury(newTreasury)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L110](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L110)


- File: contracts/GeVault.sol
```
 
Line: 131          emit PushTick(tr)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L131](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L131)


- File: contracts/GeVault.sol
```
 
Line: 160          emit ShiftTick(tr)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L160](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L160)


- File: contracts/GeVault.sol
```
 
Line: 172          emit ModifyTick(tr, index)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L172](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L172)


- File: contracts/GeVault.sol
```
 
Line: 186          emit SetFee(newBaseFeeX4)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L186](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L186)


- File: contracts/GeVault.sol
```
 
Line: 193          emit SetTvlCap(newTvlCap)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L193](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L193)


- File: contracts/GeVault.sol
```
 
Line: 362          emit Rebalance(tickIndex)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362)


- File: contracts/GeVault.sol
```
 
Line: 240          emit Withdraw(msg.sender, token, amount, liquidity)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L240](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L240)


- File: contracts/GeVault.sol
```
 
Line: 283          emit Deposit(msg.sender, token, amount, liquidity)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L283](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L283)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 323          emit ClosePosition(user, debtAsset, debt, amt0, amt1)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L323](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L323)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 148          emit BuyOptions(user, flashAsset, flashAmount, amount0, amount1)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L148](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L148)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 455          emit Swap(msg.sender, path[0], amount, path[1], received)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L455](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L455)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 106          emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1])
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L106](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L106)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 240          emit ReducedPosition(user, debtAsset, debt)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L240](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L240)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 401          emit SellOptions(msg.sender, optionAddress, deposited, amount0, amount1 )
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L401](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L401)


- File: contracts/RangeManager.sol
```
 
Line: 87          emit AddRange(startX10, endX10, tokenisedRanges.length - 1)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L87](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L87)


- File: contracts/RangeManager.sol
```
 
Line: 132          emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L132](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L132)


- File: contracts/RangeManager.sol
```
 
Line: 120          emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L120](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L120)


- File: contracts/RangeManager.sol
```
 
Line: 164          emit Deposit(msg.sender, address(tr), lpAmt)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L164](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L164)


- File: contracts/RoeRouter.sol
```
 
Line: 50          emit DeprecatePool(poolId)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L50](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L50)


- File: contracts/RoeRouter.sol
```
 
Line: 78          emit AddPool(poolId, lendingPoolAddressProvider)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L78](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L78)


- File: contracts/RoeRouter.sol
```
 
Line: 86          emit UpdateTreasury(newTreasury)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L86](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L86)


- File: contracts/TokenisableRange.sol
```
 
Line: 119          emit InitTR(address(asset0), address(asset1), startX10, endX10)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L119](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L119)


- File: contracts/TokenisableRange.sol
```
 
Line: 162          emit Deposit(msg.sender, 1e18)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L162](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L162)


- File: contracts/TokenisableRange.sol
```
 
Line: 214          emit ClaimFees(newFee0, newFee1)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L214](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L214)


- File: contracts/TokenisableRange.sol
```
 
Line: 284          emit Deposit(msg.sender, lpAmt)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L284](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L284)


- File: contracts/TokenisableRange.sol
```
 
Line: 326          emit Withdraw(msg.sender, lp)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L326](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L326)


- File: contracts/helper/FixedOracle.sol
```
 
Line: 26          emit SetHardcodedPrice(_hardcodedPrice)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L26](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L26)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 121          emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1])
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L121](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L121)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 134          emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1])
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L134](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L134)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 143          emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1])
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L143](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L143)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 157          emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1])
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L157](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L157)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 175          emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1])
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L175](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L175)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 193          emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1])
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L193](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L193)


</details>

# 


## Assembly optimization for low level calls
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 744

### Description
Identifies situations where returnData is copied to memory even if the variable is not utilized. In such cases, a low level assembly call could be a more efficient approach."
```solidity
// before
(bool success,) = payable(receiver).call{gas: gas, value: value}("");

//after
bool success;
assembly {
    success := call(gas, receiver, value, 0, 0, 0, 0)
}
```


<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/helper/V3Proxy.sol
```
 
Line: 156          msg.sender.call{value: msg.value - amounts[0]}("")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 174          payable(msg.sender).call{value: amountOut}("")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L174](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L174)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 192          payable(msg.sender).call{value: amounts[1]}("")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L192](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L192)


</details>

# 


## Division operations between unsigned could be unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 170

### Description
Division operations on unsigned integers should be unchecked to save gas since they cannot overflow or underflow. Because unsigned integers cannot have negative values, execution of division operations outside `unchecked` blocks adds nothing but overhead. Saves about 85 gas.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/TokenisableRange.sol
```
 
Line: 71          z = (x / z + z) / 2
```
 Should be unchecked - x / z.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L71](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L71)


- File: contracts/helper/LPOracle.sol
```
 
Line: 49          z = (x / z + z) / 2
```
 Should be unchecked - x / z.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L49](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L49)


</details>

# 


## Modulus operations that could be unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 255

### Description
Modulus operations should be unchecked to save gas since they cannot overflow or underflow. Execution of modulus operations outside `unchecked` blocks adds nothing but overhead. Saves about 30 gas.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/TokenisableRange.sol
```
 
Line: 112          (_lowerTick + int24(feeTier)) % int24(feeTier * 2)
```
 should be unchecked

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L112](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L112)


- File: contracts/TokenisableRange.sol
```
 
Line: 113          (_upperTick + int24(feeTier)) % int24(feeTier * 2)
```
 should be unchecked

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L113](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L113)


- File: contracts/TokenisableRange.sol
```
 
Line: 106          (midleTick + int24(feeTier)) % int24(feeTier * 2)
```
 should be unchecked

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L106](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L106)


</details>

# 



## Constants Variable Should Be Private for Gas Optimization
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 20400

### Description
This detector flags contracts that inefficiently use `public` for constant variables. Using `private` for constant variables is cheaper in terms of gas usage. If the value of the constant variable is needed, it can be accessed via a getter function. In case of multiple constant variables, a single getter function could be used to return a tuple of the values of all currently-private constants.

<details>

<summary>
There are 6 instances of this issue:

</summary>

###
- File: contracts/GeVault.sol
```
 
Line: 62          uint public constant nearbyRanges = 2
```
 `nearbyRanges` is a public. Consider making it private to save gas.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L62](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L62)


- File: contracts/RangeManager.sol
```
 
Line: 36          INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88)
```
 `POS_MGR` is a public. Consider making it private to save gas.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36)


- File: contracts/TokenisableRange.sol
```
 
Line: 58          INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88)
```
 `POS_MGR` is a public. Consider making it private to save gas.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L58](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L58)


- File: contracts/TokenisableRange.sol
```
 
Line: 59          IUniswapV3Factory constant public V3_FACTORY = IUniswapV3Factory(0x1F98431c8aD98523631AE4a59f267346ea31F984)
```
 `V3_FACTORY` is a public. Consider making it private to save gas.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L59](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L59)


- File: contracts/TokenisableRange.sol
```
 
Line: 60          address constant public treasury = 0x22Cc3f665ba4C898226353B672c5123c58751692
```
 `treasury` is a public. Consider making it private to save gas.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L60](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L60)


- File: contracts/TokenisableRange.sol
```
 
Line: 61          uint constant public treasuryFee = 20
```
 `treasuryFee` is a public. Consider making it private to save gas.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L61](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L61)


</details>

# 


## Use of emit inside a loop
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 3078

### Description
Emitting an event inside a loop performs a LOG op N times, where N is the loop length. This can lead to significant gas costs.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 148          emit BuyOptions(user, flashAsset, flashAmount, amount0, amount1)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L148](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L148)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 323          emit ClosePosition(user, debtAsset, debt, amt0, amt1)
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L323](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L323)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 106          emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1])
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L106](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L106)


</details>

# 


## Identical Deployments Should be Replaced with Clone
- Severity: Gas Optimization
- Confidence: High

### Description
Detects instances where the same contract is deployed multiple times. Such cases might benefit from the clone factory pattern (EIP-1167), which can significantly reduce deployment gas costs.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
-     - BeaconProxy is deployed in multiple functions:
File: contracts/RangeManager.sol
```
 
Line: 79          BeaconProxy trbp = new BeaconProxy(beacon, "")
```
File: contracts/RangeManager.sol
```
 
Line: 81          trbp = new BeaconProxy(beacon, "")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L79](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L79)


</details>

# 


## Optimal State Variable Order
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 5000

### Description
Detects optimal variable order in contract storage layouts to decrease the number of storage slots used. Each storage slot used can cost at least 5,000 gas for each write operation, and potentially up to 20,000 gas if you're turning a zero value into a non-zero value. Hence, optimizing storage usage can result in significant gas savings. The real-world savings could vary depending on your contract's specific logic and the state of the Ethereum network.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- Optimization opportunity in storage variable layout of contract File: contracts/TokenisableRange.sol
```
 
Line: 18          contract TokenisableRange is ERC20("", ""), ReentrancyGuard 
```

- original variable order (count: 17 slots)
    - int24 lowerTick
    - int24 upperTick
    - uint24 feeTier
    - uint256 tokenId
    - uint256 fee0
    - uint256 fee1
    - TokenisableRange.ASSET TOKEN0
    - TokenisableRange.ASSET TOKEN1
    - IAaveOracle ORACLE
    - string _name
    - string _symbol
    - TokenisableRange.ProxyState status
    - address creator
    - uint128 liquidity
    - address TREASURY_DEPRECATED
    - uint256 treasuryFee_deprecated
    - INonfungiblePositionManager POS_MGR
    - IUniswapV3Factory V3_FACTORY
    - address treasury
    - uint256 treasuryFee


 - optimized variable order (count: 16 slots)
    - IAaveOracle ORACLE
    - int24 lowerTick
    - int24 upperTick
    - uint24 feeTier
    - TokenisableRange.ProxyState status
    - address creator
    - address TREASURY_DEPRECATED
    - INonfungiblePositionManager POS_MGR
    - IUniswapV3Factory V3_FACTORY
    - address treasury
    - uint128 liquidity
    - uint256 tokenId
    - uint256 fee0
    - uint256 fee1
    - TokenisableRange.ASSET TOKEN0
    - TokenisableRange.ASSET TOKEN1
    - string _name
    - string _symbol
    - uint256 treasuryFee_deprecated
    - uint256 treasuryFee



Recomended optimizations will use 1 less slots.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L18-L384](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L18-L384)


</details>

# 


## Unused Named Return Variables
- Severity: Gas Optimization
- Confidence: High

### Description
Named return variables allow for clear and explicit naming of values to be returned from a function. However, when these variables are unused, it can lead to confusion and make the code less maintainable.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/TokenisableRange.sol
```
 
Line: 352          function getValuePerLPAtPrice(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 priceX1e8) 
```
there is not use of this variables:
@ **priceX1e8**

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352-L357](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352-L357)


- File: contracts/TokenisableRange.sol
```
 
Line: 361          function latestAnswer() public view returns (uint256 priceX1e8) 
```
there is not use of this variables:
@ **priceX1e8**

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L361-L363](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L361-L363)


- File: contracts/helper/OracleConvert.sol
```
 
Line: 78          function latestRoundData() external view returns (uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound) 
```
there is not use of this variables:
@ **roundId**
@ **answer**
@ **startedAt**
@ **updatedAt**
@ **answeredInRound**

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L78-L80](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L78-L80)


</details>

# 


## Unused state variable
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 2900

### Description
Unused state variable.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/GeVault.sol
```
 
Line: 41          RangeManager rangeManager
```
 is never used in File: contracts/GeVault.sol
```
 
Line: 27          contract GeVault is ERC20, Ownable, ReentrancyGuard 
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L41](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L41)


</details>

# 


## Safe Subtraction Should Be Unchecked
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 595

### Description
This detector identifies subtraction operations where the subtraction could safely be unchecked. This occurs when a prior if-statement or require() function ensures that the first operand (minuend) is greater than or equal to the second operand (subtrahend). In these cases, an unchecked subtraction would not result in an underflow error, and can save gas by skipping the redundant check.

<details>

<summary>
There are 7 instances of this issue:

</summary>

###
- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 298          swapTokensForExactTokens(ammRouter, token0Amount - amtA, amtB, path)
```
 should be unchecked because there is check:<br>File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 295          amtA < token0Amount
```
<br><br>
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L298](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L298)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 303          swapTokensForExactTokens(ammRouter, token1Amount - amtB, amtA, path)
```
 should be unchecked because there is check:<br>File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 300          amtB < token1Amount
```
<br><br>
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L303](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L303)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 316          amt0 = amtA - amt0
```
 should be unchecked because there is check:<br>File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 315          amtA > amt0
```
<br><br>
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L316](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L316)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 320          amt1 = amtB - amt1
```
 should be unchecked because there is check:<br>File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 319          amtB > amt1
```
<br><br>
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L320](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L320)


- File: contracts/PositionManager/PositionManager.sol
```
 
Line: 143          valueA = valueA / 10 ** (decimalsA - decimalsB)
```
 should be unchecked because there is check:<br>File: contracts/PositionManager/PositionManager.sol
```
 
Line: 143          decimalsA > decimalsB
```
<br><br>
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L143](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L143)


- File: contracts/PositionManager/PositionManager.sol
```
 
Line: 144          valueB = valueB / 10 ** (decimalsB - decimalsA)
```
 should be unchecked because there is check:<br>File: contracts/PositionManager/PositionManager.sol
```
 
Line: 144          decimalsA < decimalsB
```
<br><br>
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L144](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L144)


- File: contracts/helper/LPOracle.sol
```
 
Line: 90          norm_b = sqrt( a * b * priceA * 10**(decimalsB-decimalsA) / priceB )
```
 should be unchecked because there is check:<br>File: contracts/helper/LPOracle.sol
```
 
Line: 89          decimalsB >= decimalsA
```
<br><br>
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L90](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L90)


</details>

# 


## Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/RangeManager.sol
```
 
Line: 83          IAaveOracle oracle = IAaveOracle(ILendingPoolAddressesProvider( LENDING_POOL.getAddressesProvider() ).getPriceOracle())
```
Unnecessary cast: `ILendingPoolAddressesProvider( LENDING_POOL.getAddressesProvider() )` it cast to the same type.<br>
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L83](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L83)


</details>

# 




## Usage of Custom Errors for Gas Efficiency
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 22080

### Description
This detector flags functions that use revert()/require() strings, which are less gas efficient than custom errors. Custom errors, available from Solidity version 0.8.4, save approximately 50 gas each time they're used by avoiding the need to allocate and store the revert string.

<details>

<summary>
There are 80 instances of this issue:

</summary>

###
- File: contracts/GeVault.sol
```
 
Line: 79          require(_treasury != address(0x0), "GEV: Invalid Treasury")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L79](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L79)


- File: contracts/GeVault.sol
```
 
Line: 80          require(_uniswapPool != address(0x0), "GEV: Invalid Pool")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L80](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L80)


- File: contracts/GeVault.sol
```
 
Line: 81          require(weth != address(0x0), "GEV: Invalid WETH")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L81](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L81)


- File: contracts/GeVault.sol
```
 
Line: 120          require(t0 == token0 && t1 == token1, "GEV: Invalid TR")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120)


- File: contracts/GeVault.sol
```
 
Line: 125          require( t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L125](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L125)


- File: contracts/GeVault.sol
```
 
Line: 127          require( t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L127](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L127)


- File: contracts/GeVault.sol
```
 
Line: 141          require(t0 == token0 && t1 == token1, "GEV: Invalid TR")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L141](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L141)


- File: contracts/GeVault.sol
```
 
Line: 146          require( t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L146](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L146)


- File: contracts/GeVault.sol
```
 
Line: 148          require( t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L148](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L148)


- File: contracts/GeVault.sol
```
 
Line: 170          require(t0 == token0 && t1 == token1, "GEV: Invalid TR")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L170](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L170)


- File: contracts/GeVault.sol
```
 
Line: 184          require(newBaseFeeX4 < 1e4, "GEV: Invalid Base Fee")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L184](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L184)


- File: contracts/GeVault.sol
```
 
Line: 203          require(poolMatchesOracle(), "GEV: Oracle Error")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L203](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L203)


- File: contracts/GeVault.sol
```
 
Line: 215          require(poolMatchesOracle(), "GEV: Oracle Error")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L215](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L215)


- File: contracts/GeVault.sol
```
 
Line: 217          require(liquidity <= balanceOf(msg.sender), "GEV: Insufficient Balance")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L217](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L217)


- File: contracts/GeVault.sol
```
 
Line: 218          require(liquidity > 0, "GEV: Withdraw Zero")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L218](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L218)


- File: contracts/GeVault.sol
```
 
Line: 249          require(isEnabled, "GEV: Pool Disabled")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L249](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L249)


- File: contracts/GeVault.sol
```
 
Line: 250          require(poolMatchesOracle(), "GEV: Oracle Error")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L250](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L250)


- File: contracts/GeVault.sol
```
 
Line: 251          require(token == address(token0) || token == address(token1), "GEV: Invalid Token")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L251](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L251)


- File: contracts/GeVault.sol
```
 
Line: 252          require(amount > 0 || msg.value > 0, "GEV: Deposit Zero")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L252](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L252)


- File: contracts/GeVault.sol
```
 
Line: 256          require(token == address(WETH), "GEV: Invalid Weth")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L256](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L256)


- File: contracts/GeVault.sol
```
 
Line: 269          require(tvlCap > valueX8 + getTVL(), "GEV: Max Cap Reached")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L269](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L269)


- File: contracts/GeVault.sol
```
 
Line: 281          require(liquidity > 0, "GEV: No Liquidity Added")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L281](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L281)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 69          require( address(lendingPool) == msg.sender, "OPM: Call Unallowed")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L69](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L69)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 91          require( address(lendingPool) == msg.sender, "OPM: Call Unallowed")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L91](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L91)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 137          require(sourceSwap == token0 || sourceSwap == token1, "OPM: Invalid Swap Token")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L137](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L137)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 167          require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L167](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L167)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 198          require(options.length == amounts.length, "ARRAY_LEN_MISMATCH")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L198](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L198)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 235          require(debt > 0, "OPM: No Debt")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L235](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L235)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 261          require(collateralAsset == token0 || collateralAsset == token1, "OPM: Invalid Collateral Asset")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L261](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L261)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 344          require( 
      (debtValue < 1e8 && tokensValue < 1e8 )
      || (tokensValue > debtValue * 98 / 100 && tokensValue < debtValue * 102 / 100), 
      "OPM: Slippage Error"
    )
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L344-L348](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L344-L348)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 369          require(feeAmount <= IERC20(collateralAsset).balanceOf(address(this)), "OPM: Insufficient Collateral")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L369](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L369)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 393          require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" )
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L393](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L393)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 420          require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" )
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L420](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L420)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 441          require(sourceAsset == token0 || sourceAsset == token1, "OPM: Invalid Swap Asset")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L441](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L441)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 495          require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" )
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L495](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L495)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 496          require( amountsIn[0] <= ERC20(path[0]).balanceOf(address(this)), "OPM: Insufficient Token Amount" )
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L496](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L496)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 523          require ( priceAssetA > 0 && priceAssetB > 0, "OPM: Invalid Oracle Price")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L523](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L523)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 525          require( amountB > 0, "OPM: Target Amount Too Low")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L525](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L525)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 536          require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L536](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L536)


- File: contracts/PositionManager/PositionManager.sol
```
 
Line: 30          require(roerouter != address(0x0), "Invalid address")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L30](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L30)


- File: contracts/PositionManager/PositionManager.sol
```
 
Line: 145          require( valueA <= valueB * 101 / 100, "PM: LP Oracle Error")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L145](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L145)


- File: contracts/PositionManager/PositionManager.sol
```
 
Line: 146          require( valueB <= valueA * 101 / 100, "PM: LP Oracle Error")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L146](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L146)


- File: contracts/RangeManager.sol
```
 
Line: 49          require( address(lendingPool) != address(0x0), "Invalid address" )
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L49](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L49)


- File: contracts/RangeManager.sol
```
 
Line: 60          require(start < end, "Range invalid")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L60](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L60)


- File: contracts/RangeManager.sol
```
 
Line: 66          revert("Range overlap")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L66](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L66)


- File: contracts/RangeManager.sol
```
 
Line: 76          require(beacon != address(0x0), "Invalid beacon")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L76](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L76)


- File: contracts/RangeManager.sol
```
 
Line: 108          require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L108](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L108)


- File: contracts/RangeManager.sol
```
 
Line: 206          require(hf > 1.01e18, "Health factor is too low")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L206](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L206)


- File: contracts/RoeRouter.sol
```
 
Line: 34          require(treasury_ != address(0x0), "Invalid address")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L34](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L34)


- File: contracts/RoeRouter.sol
```
 
Line: 68          require (
      lendingPoolAddressProvider != address(0x0) 
      && token0 != address(0x0) 
      && token1 != address(0x0) 
      && ammRouter != address(0x0), 
      "Invalid Address"
    )
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L68-L74](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L68-L74)


- File: contracts/RoeRouter.sol
```
 
Line: 75          require(token0 < token1, "Invalid Order")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L75](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L75)


- File: contracts/RoeRouter.sol
```
 
Line: 84          require(newTreasury != address(0x0), "Invalid address")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L84](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L84)


- File: contracts/TokenisableRange.sol
```
 
Line: 86          require(address(_oracle) != address(0x0), "Invalid oracle")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L86](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L86)


- File: contracts/TokenisableRange.sol
```
 
Line: 87          require(status == ProxyState.INIT_PROXY, "!InitProxy")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L87](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L87)


- File: contracts/TokenisableRange.sol
```
 
Line: 135          require(status == ProxyState.INIT_LP, "!InitLP")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L135](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L135)


- File: contracts/TokenisableRange.sol
```
 
Line: 136          require(msg.sender == creator, "Unallowed call")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L136](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L136)


- File: contracts/TokenisableRange.sol
```
 
Line: 209          require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L209](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L209)


- File: contracts/TokenisableRange.sol
```
 
Line: 225          require(totalSupply() > 0, "TR Closed")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L225](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L225)


- File: contracts/TokenisableRange.sol
```
 
Line: 271          require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L271](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L271)


- File: contracts/helper/FixedOracle.sol
```
 
Line: 11          require(msg.sender == owner, "Only the owner can call this function.")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11)


- File: contracts/helper/LPOracle.sol
```
 
Line: 29          require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L29](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L29)


- File: contracts/helper/LPOracle.sol
```
 
Line: 63          require(timeStamp > 0, "Round not complete")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L63](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L63)


- File: contracts/helper/LPOracle.sol
```
 
Line: 101          require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L101](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L101)


- File: contracts/helper/OracleConvert.sol
```
 
Line: 23          require(clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L23](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L23)


- File: contracts/helper/OracleConvert.sol
```
 
Line: 26          require(CL_TOKENA.decimals() + CL_TOKENB.decimals() >= 16, "Decimals error")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L26](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L26)


- File: contracts/helper/OracleConvert.sol
```
 
Line: 44          require(timeStamp > 0, "Round not complete")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L44](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L44)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 87          require(acceptPayable, "CannotReceiveETH")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L87](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L87)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 91          require(acceptPayable, "CannotReceiveETH")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L91](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L91)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 99          require(path.length == 2, "Direct swap only")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L99](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L99)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 106          require(path.length == 2, "Direct swap only")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L106](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L106)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 113          require(path.length == 2, "Direct swap only")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L113](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L113)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 125          require(path.length == 2, "Direct swap only")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L125](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L125)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 138          require(path.length == 2, "Direct swap only")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L138](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L138)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 139          require(path[0] == ROUTER.WETH9(), "Invalid path")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L139](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L139)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 148          require(path.length == 2, "Direct swap only")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L148](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L148)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 149          require(path[0] == ROUTER.WETH9(), "Invalid path")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L149](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L149)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 161          require(path.length == 2, "Direct swap only")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L161](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L161)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 162          require(path[1] == ROUTER.WETH9(), "Invalid path")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L162](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L162)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 179          require(path.length == 2, "Direct swap only")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L179](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L179)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 180          require(path[1] == ROUTER.WETH9(), "Invalid path")
```
 use custom error instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L180](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L180)


</details>

# 




## Greater or Equal Comparison Costs Less Gas than Greater Comparison
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 15

### Description
In Solidity, the >= operator costs less gas than the > operator. This is because the Solidity compiler uses the LT opcode for >= comparisons, which requires less gas than the combination of GT and ISZERO opcodes used for > comparisons. Therefore, replacing > with >= when possible can reduce the gas cost of your contract.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 315          amtA > amt0
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L315](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L315)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 319          amtB > amt1
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L319](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L319)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 549          scale0 > scale1
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L549](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L549)


- File: contracts/RangeManager.sol
```
 
Line: 51          ASSET_0 = _asset0 < _asset1 ? _asset0 : _asset1
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L51](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L51)


- File: contracts/RangeManager.sol
```
 
Line: 52          ASSET_1 = _asset0 < _asset1 ? _asset1 : _asset0
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L52](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L52)


</details>

# 


## Inefficient Parameter Storage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 100

### Description
When passing function parameters, using the `calldata` area instead of `memory` can improve gas efficiency. Calldata is a read-only area where function arguments and external function calls' parameters are stored.

By using `calldata` for function parameters, you avoid unnecessary gas costs associated with copying data from `calldata` to memory. This is particularly beneficial when the parameter is read-only and doesn't require modification within the contract.

Using `calldata` for function parameters can help optimize gas usage, especially when making external function calls or when the parameter values are provided externally and don't need to be stored persistently within the contract.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: interfaces/PoolAddress.sol
```
 
Line: 33          PoolKey memory key
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L33](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L33)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 66          address[] memory sourceSwap
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L66](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L66)


</details>

# 


## Short-circuit rules can be used to optimize some gas usage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 4200

### Description
Some conditions may be reordered to save an SLOAD (2100 gas), as we avoid reading state variables when the first part of the condition fails (with &&), or succeeds (with ||). For instance, consider a scenario where you have a `stateVariable` (a variable stored in contract storage) and a `localVariable` (a variable in memory). 

If you have a condition like `stateVariable > 0 && localVariable > 0`, if `localVariable > 0` is false, the Solidity runtime will still execute `stateVariable > 0`, which costs an SLOAD operation (2100 gas). However, if you reorder the condition to `localVariable > 0 && stateVariable > 0`, the `stateVariable > 0` check won't happen if `localVariable > 0` is false, saving you the SLOAD gas cost.

Similarly, for the `||` operator, if you have a condition like `stateVariable > 0 || localVariable > 0`, and `stateVariable > 0` is true, the Solidity runtime will still execute `localVariable > 0`. But if you reorder the condition to `localVariable > 0 || stateVariable > 0`, and `localVariable > 0` is true, the `stateVariable > 0` check won't happen, again saving you the SLOAD gas cost.

This detector checks for such conditions in the contract and reports if any condition could be optimized by taking advantage of the short-circuiting behavior of `&&` and `||`.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/TokenisableRange.sol
```
 
Line: 237          fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 )
```


```
 // @audit: Switch (n0 > 0 || fee0 == 0) && fee0 + fee1 > 0 
```
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L237](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L237)


- File: contracts/TokenisableRange.sol
```
 
Line: 237          fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 )
```


```
 // @audit: Switch (n1 > 0 || fee1 == 0) && fee0 + fee1 > 0 && (n0 > 0 || fee0 == 0) 
```
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L237](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L237)


</details>

# 