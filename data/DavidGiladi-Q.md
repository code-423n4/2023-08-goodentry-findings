
### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] Array out of bounds accesses | [Array out of bounds accesses](#array-out-of-bounds-accesses) | 5 |
|[L-2] Burn functions must be protected with a modifier | [Burn functions must be protected with a modifier](#burn-functions-must-be-protected-with-a-modifier) | 3 |
|[L-3] Missing contract-existence checks before low-level calls | [Missing contract-existence checks before low-level calls](#missing-contract-existence-checks-before-low-level-calls) | 3 |
|[L-4] Local variable shadowing | [Local variable shadowing](#local-variable-shadowing) | 4 |
|[L-5] Missing gap Storage Variable in Upgradeable Contract | [Missing gap Storage Variable in Upgradeable Contract](#missing-gap-storage-variable-in-upgradeable-contract) | 2 |
|[L-6] Reentrancy vulnerabilities | [Reentrancy vulnerabilities](#reentrancy-vulnerabilities) | 10 |
|[L-7] Restrictive casts in assignments | [Restrictive casts in assignments](#restrictive-casts-in-assignments) | 3 |

Total: 7 issues


### Non-Critical Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[N-1] Dead-code | [Dead-code](#dead-code) | 3 |
|[N-2] Duplicated require()/revert() checks should be refactored to a modifier or function | [Duplicated require()/revert() checks should be refactored to a modifier or function](#duplicated-requirerevert-checks-should-be-refactored-to-a-modifier-or-function) | 9 |
|[N-3] Lack of Explicit Visibility Declaration in State Variables | [Lack of Explicit Visibility Declaration in State Variables](#lack-of-explicit-visibility-declaration-in-state-variables) | 4 |
|[N-4] Inconsistent usage of require/error | [Inconsistent usage of require/error](#inconsistent-usage-of-requireerror) | 1 |
|[N-5] Max Value Optimization Replacing 2^n - 1 with type(uint n).max | [Max Value Optimization Replacing 2^n - 1 with type(uint n).max](#max-value-optimization-replacing-2n---1-with-typeuint-nmax) | 4 |
|[N-6] Token contract should have a blacklist function or modifier | [Token contract should have a blacklist function or modifier](#token-contract-should-have-a-blacklist-function-or-modifier) | 2 |
|[N-7] Missing inheritance | [Missing inheritance](#missing-inheritance) | 3 |
|[N-8] Name reused | [Name reused](#name-reused) | 1 |
|[N-9] Override function arguments that are unused should have the variable name removed or commented out | [Override function arguments that are unused should have the variable name removed or commented out](#override-function-arguments-that-are-unused-should-have-the-variable-name-removed-or-commented-out) | 2 |
|[N-10] Variable names too similar | [Variable names too similar](#variable-names-too-similar) | 41 |
|[N-11] Single Source of Constants Ensuring Consistency and Integrity | [Single Source of Constants Ensuring Consistency and Integrity](#single-source-of-constants-ensuring-consistency-and-integrity) | 1 |

Total: 1 issues

## Array out of bounds accesses
- Severity: Low
- Confidence: Medium

### Description
If the lengths of arrays are not checked before they're accessed, user operations may not be executed properly due to the potential issue of accessing an array out of its bounds. In Solidity, accessing an array beyond its boundaries can cause a runtime error known as an out-of-bounds exception. This error arises when attempting to read or write to an element of an array that does not exist. Therefore, it is critical to avoid out-of-bounds exceptions while accessing arrays in Solidity code to prevent unexpected halts and possible loss of funds.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 64          uint256[] calldata amounts
```
 is accessed on File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 73          uint amount = amounts[k]
```
 index that might be out-of-bounds

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L64](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L64)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 66          address[] memory sourceSwap
```
 is accessed on File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 74          withdrawOptionAssets(poolId, asset, amount, sourceSwap[k], user)
```
 index that might be out-of-bounds

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L66](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L66)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 86          uint256[] calldata amounts
```
 is accessed on File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 97          uint amount = amounts[k]
```
 index that might be out-of-bounds

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L86](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L86)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 470          address[] memory path
```
 is accessed on File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 474          checkSetAllowance(path[0], address(ammRouter), amount)
```
 index that might be out-of-bounds

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L470](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L470)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 491          address[] memory path
```
 is accessed on File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 492          checkSetAllowance(path[0], address(ammRouter), maxAmount)
```
 index that might be out-of-bounds

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L491](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L491)


</details>

# 



## Missing contract-existence checks before low-level calls
- Severity: Low
- Confidence: High

### Description
Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0` or use the `extcodesize` assembly operation to verify the presence of contract code at the specified address. Both these methods ensure the existence of a contract before making a low-level call.

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


## Burn functions must be protected with a modifier
- Severity: Low
- Confidence: High

### Description
If burn functions are not protected by a modifier, any address may be able to burn tokens, potentially leading to financial loss. A common modifier to use is `onlyOwner`.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: interfaces/IAToken.sol
```
 
Line: 61          function burn(
    address user,
    address receiverOfUnderlying,
    uint256 amount,
    uint256 index
  ) external;
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAToken.sol#L61-L66](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAToken.sol#L61-L66)


- File: interfaces/IUniswapV2Pair.sol
```
 
Line: 46          function burn(address to) external returns (uint amount0, uint amount1);
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV2Pair.sol#L46](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV2Pair.sol#L46)


- File: interfaces/INonfungiblePositionManager.sol
```
 
Line: 179          function burn(uint256 tokenId) external payable;
```
Burn function without a protective modifier.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L179](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L179)


</details>

# 


## Missing gap Storage Variable in Upgradeable Contract
- Severity: Low
- Confidence: High

### Description
upgradeable contracts that are missing a '__gap' storage variable. In upgradeable contracts, it is important to reserve storage slots for future versions to introduce new storage variables. The '__gap' storage variable acts as a placeholder, allowing for seamless upgrades without affecting existing storage layout.When a contract is not designed with a '__gap' storage variable, adding new storage variables in subsequent versions becomes problematic. It can lead to storage collisions or layout incompatibilities, making it difficult to upgrade the contract without requiring costly data migrations or redeployments.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: interfaces/IAToken.sol
```
 
Line: 9          interface IAToken is IERC20, IScaledBalanceToken, IInitializableAToken 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAToken.sol#L9-L112](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAToken.sol#L9-L112)


- File: interfaces/IInitializableAToken.sol
```
 
Line: 12          interface IInitializableAToken 
```
 missing __gap storage variable

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IInitializableAToken.sol#L12-L55](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IInitializableAToken.sol#L12-L55)


</details>

# 



## Local variable shadowing
- Severity: Low
- Confidence: High

### Description
Detection of shadowing using local variables.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: interfaces/IAaveIncentivesController.sol
```
 
Line: 67          address[] calldata assets
```
 shadows:
	- File: interfaces/IAaveIncentivesController.sol
```
 
Line: 39          function assets(address asset)
    external
    view
    returns (
      uint128,
      uint128,
      uint256
    );
```
 (function)

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L67](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L67)


- File: interfaces/IAaveIncentivesController.sol
```
 
Line: 87          address[] calldata assets
```
 shadows:
	- File: interfaces/IAaveIncentivesController.sol
```
 
Line: 39          function assets(address asset)
    external
    view
    returns (
      uint128,
      uint128,
      uint256
    );
```
 (function)

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L87](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L87)


- File: interfaces/IAaveIncentivesController.sol
```
 
Line: 99          address[] calldata assets
```
 shadows:
	- File: interfaces/IAaveIncentivesController.sol
```
 
Line: 39          function assets(address asset)
    external
    view
    returns (
      uint128,
      uint128,
      uint256
    );
```
 (function)

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L99](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L99)


- File: interfaces/IAaveIncentivesController.sol
```
 
Line: 113          address[] calldata assets
```
 shadows:
	- File: interfaces/IAaveIncentivesController.sol
```
 
Line: 39          function assets(address asset)
    external
    view
    returns (
      uint128,
      uint128,
      uint256
    );
```
 (function)

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L113](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L113)


</details>

# 



## Reentrancy vulnerabilities
- Severity: Low
- Confidence: Medium

### Description

Detection of the [reentrancy bug](https://github.com/trailofbits/not-so-smart-contracts/tree/master/reentrancy).
Only report reentrancy that acts as a double call (see `reentrancy-eth`, `reentrancy-no-eth`).

<details>

<summary>
There are 10 instances of this issue:

</summary>

###
- Reentrancy in File: contracts/RangeManager.sol
```
 
Line: 75          function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner 
```
:
	External calls:
	- File: contracts/RangeManager.sol
```
 
Line: 79          BeaconProxy trbp = new BeaconProxy(beacon, "")
```

	State variables written after the call(s):
	- File: contracts/RangeManager.sol
```
 
Line: 80          tokenisedRanges.push( TokenisableRange(address(trbp)) )
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75-L88](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75-L88)


- Reentrancy in File: contracts/RangeManager.sol
```
 
Line: 75          function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner 
```
:
	External calls:
	- File: contracts/RangeManager.sol
```
 
Line: 79          BeaconProxy trbp = new BeaconProxy(beacon, "")
```

	- File: contracts/RangeManager.sol
```
 
Line: 81          trbp = new BeaconProxy(beacon, "")
```

	State variables written after the call(s):
	- File: contracts/RangeManager.sol
```
 
Line: 82          tokenisedTicker.push( TokenisableRange(address(trbp)) )
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75-L88](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75-L88)


- Reentrancy in File: contracts/TokenisableRange.sol
```
 
Line: 167          function claimFee() public 
```
:
	External calls:
	- File: contracts/TokenisableRange.sol
```
 
Line: 168          (uint256 newFee0, uint256 newFee1) = POS_MGR.collect( 
      INonfungiblePositionManager.CollectParams({
        tokenId: tokenId,
        recipient: address(this),
        amount0Max: type(uint128).max,
        amount1Max: type(uint128).max
      })
    )
```

	- File: contracts/TokenisableRange.sol
```
 
Line: 180          TOKEN0.token.safeTransfer(treasury, tf0)
```

	- File: contracts/TokenisableRange.sol
```
 
Line: 181          TOKEN1.token.safeTransfer(treasury, tf1)
```

	State variables written after the call(s):
	- File: contracts/TokenisableRange.sol
```
 
Line: 183          fee0 = fee0 + newFee0 - tf0
```

	- File: contracts/TokenisableRange.sol
```
 
Line: 184          fee1 = fee1 + newFee1 - tf1
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L167-L215](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L167-L215)


- Reentrancy in File: contracts/TokenisableRange.sol
```
 
Line: 134          function init(uint n0, uint n1) external 
```
:
	External calls:
	- File: contracts/TokenisableRange.sol
```
 
Line: 138          TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0)
```

	- File: contracts/TokenisableRange.sol
```
 
Line: 139          TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1)
```

	- File: contracts/TokenisableRange.sol
```
 
Line: 140          TOKEN0.token.safeIncreaseAllowance(address(POS_MGR), n0)
```

	- File: contracts/TokenisableRange.sol
```
 
Line: 141          TOKEN1.token.safeIncreaseAllowance(address(POS_MGR), n1)
```

	- File: contracts/TokenisableRange.sol
```
 
Line: 142          (tokenId, liquidity, , ) = POS_MGR.mint( 
      INonfungiblePositionManager.MintParams({
         token0: address(TOKEN0.token),
         token1: address(TOKEN1.token),
         fee: feeTier * 100,
         tickLower: lowerTick,
         tickUpper: upperTick,
         amount0Desired: n0,
         amount1Desired: n1,
         amount0Min: n0 * 95 / 100,
         amount1Min: n1 * 95 / 100,
         recipient: address(this),
         deadline: block.timestamp
      })
    )
```

	State variables written after the call(s):
	- File: contracts/TokenisableRange.sol
```
 
Line: 142          (tokenId, liquidity, , ) = POS_MGR.mint( 
      INonfungiblePositionManager.MintParams({
         token0: address(TOKEN0.token),
         token1: address(TOKEN1.token),
         fee: feeTier * 100,
         tickLower: lowerTick,
         tickUpper: upperTick,
         amount0Desired: n0,
         amount1Desired: n1,
         amount0Min: n0 * 95 / 100,
         amount1Min: n1 * 95 / 100,
         recipient: address(this),
         deadline: block.timestamp
      })
    )
```

	- File: contracts/TokenisableRange.sol
```
 
Line: 142          (tokenId, liquidity, , ) = POS_MGR.mint( 
      INonfungiblePositionManager.MintParams({
         token0: address(TOKEN0.token),
         token1: address(TOKEN1.token),
         fee: feeTier * 100,
         tickLower: lowerTick,
         tickUpper: upperTick,
         amount0Desired: n0,
         amount1Desired: n1,
         amount0Min: n0 * 95 / 100,
         amount1Min: n1 * 95 / 100,
         recipient: address(this),
         deadline: block.timestamp
      })
    )
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L134-L163](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L134-L163)


- Reentrancy in File: contracts/helper/V3Proxy.sol
```
 
Line: 147          function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) 
```
:
	External calls:
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 151          amounts[0] = ROUTER.exactOutputSingle{value: msg.value}(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountOut, msg.value, 0))
```

	State variables written after the call(s):
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 153          acceptPayable = true
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L147-L158](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L147-L158)


- Reentrancy in File: contracts/helper/V3Proxy.sol
```
 
Line: 147          function swapETHForExactTokens(uint amountOut, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) 
```
:
	External calls:
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 151          amounts[0] = ROUTER.exactOutputSingle{value: msg.value}(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountOut, msg.value, 0))
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 154          ROUTER.refundETH()
```

	External calls sending eth:
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 151          amounts[0] = ROUTER.exactOutputSingle{value: msg.value}(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountOut, msg.value, 0))
```

	State variables written after the call(s):
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 155          acceptPayable = false
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L147-L158](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L147-L158)


- Reentrancy in File: contracts/helper/V3Proxy.sol
```
 
Line: 178          function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) 
```
:
	External calls:
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 182          ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 183          ogInAsset.safeApprove(address(ROUTER), amountIn)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 186          amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountIn, amountOutMin, 0))
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 187          ogInAsset.safeApprove(address(ROUTER), 0)
```

	State variables written after the call(s):
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 189          acceptPayable = true
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L178-L194](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L178-L194)


- Reentrancy in File: contracts/helper/V3Proxy.sol
```
 
Line: 178          function swapExactTokensForETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) 
```
:
	External calls:
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 182          ogInAsset.safeTransferFrom(msg.sender, address(this), amountIn)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 183          ogInAsset.safeApprove(address(ROUTER), amountIn)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 186          amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountIn, amountOutMin, 0))
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 187          ogInAsset.safeApprove(address(ROUTER), 0)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 190          weth.withdraw(amounts[1])
```

	State variables written after the call(s):
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 191          acceptPayable = false
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L178-L194](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L178-L194)


- Reentrancy in File: contracts/helper/V3Proxy.sol
```
 
Line: 160          function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) 
```
:
	External calls:
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 164          ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 165          ogInAsset.safeApprove(address(ROUTER), amountInMax)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 167          amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountOut, amountInMax, 0))
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 169          ogInAsset.safeApprove(address(ROUTER), 0)
```

	State variables written after the call(s):
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 171          acceptPayable = true
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L160-L176](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L160-L176)


- Reentrancy in File: contracts/helper/V3Proxy.sol
```
 
Line: 160          function swapTokensForExactETH(uint amountOut, uint amountInMax, address[] calldata path, address to, uint deadline) payable external returns (uint[] memory amounts) 
```
:
	External calls:
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 164          ogInAsset.safeTransferFrom(msg.sender, address(this), amountInMax)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 165          ogInAsset.safeApprove(address(ROUTER), amountInMax)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 167          amounts[0] = ROUTER.exactOutputSingle(ISwapRouter.ExactOutputSingleParams(path[0], path[1], feeTier, address(this), deadline, amountOut, amountInMax, 0))
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 169          ogInAsset.safeApprove(address(ROUTER), 0)
```

	- File: contracts/helper/V3Proxy.sol
```
 
Line: 172          weth.withdraw(amountOut)
```

	State variables written after the call(s):
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 173          acceptPayable = false
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L160-L176](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L160-L176)


</details>

# 




## Restrictive casts in assignments
- Severity: Low
- Confidence: Medium

### Description
This detector flags the instances where a restrictive cast is being used in an assignment. For example, if an address variable 'foo' is used in an expression like 'IERC20 token = FooToken(foo)', the specific cast to 'FooToken' is unnecessary as the compiler only checks that 'FooToken' extends 'IERC20' and does not check any function signatures. In such cases, it would be more appropriate to use 'IERC20 token = IERC20(foo)' or even better, 'FooToken token = FooToken(foo)'. The former allows the file in which it's used to possibly remove the import for 'FooToken', leading to cleaner and more efficient code.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/GeVault.sol
```
 
Line: 370          uint decimals0 = token0.decimals()
```

```
//  @audit: uint256 vs uint8 
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L370](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L370)


- File: contracts/GeVault.sol
```
 
Line: 371          uint decimals1 = token1.decimals()
```

```
//  @audit: uint256 vs uint8 
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L371](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L371)


- File: contracts/TokenisableRange.sol
```
 
Line: 274          feeLiquidity = newLiquidity * ( (fee0 * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (fee1 * TOKEN1_PRICE / 10 ** TOKEN1.decimals) )   
                                    / ( (added0   * TOKEN0_PRICE / 10 ** TOKEN0.decimals) + (added1   * TOKEN1_PRICE / 10 ** TOKEN1.decimals) )
```

```
//  @audit: uint256 vs uint128 
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L274-L275](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L274-L275)


</details>

# 




## Dead-code
- Severity: Non-Critical
- Confidence: Medium

### Description
Functions that are not sued.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: interfaces/PoolAddress.sol
```
 
Line: 33          function computeAddress(address factory, PoolKey memory key) internal pure returns (address pool) 
```
 is never used and should be removed

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L33-L49](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L33-L49)


- File: interfaces/PoolAddress.sol
```
 
Line: 20          function getPoolKey(
        address tokenA,
        address tokenB,
        uint24 fee
    ) internal pure returns (PoolKey memory) 
```
 is never used and should be removed

[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L20-L27](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/PoolAddress.sol#L20-L27)


- File: contracts/PositionManager/PositionManager.sol
```
 
Line: 138          function validateValuesAgainstOracle(IPriceOracle oracle, address assetA, uint amountA, address assetB, uint amountB) internal view 
```
 is never used and should be removed

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L138-L147](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L138-L147)


</details>

# 


## Duplicated require()/revert() checks should be refactored to a modifier or function
- Severity: Non-Critical
- Confidence: High

### Description
This detector checks for instances where the same require() or revert() condition is repeated in multiple places within the same contract. Such duplicate checks can often be refactored into a modifier or function, reducing code redundancy and increasing maintainability.

<details>

<summary>
There are 9 instances of this issue:

</summary>

###
- File: contracts/GeVault.sol
```
 
Line: 120          require(t0 == token0 && t1 == token1, "GEV: Invalid TR")
```
Has duplicate:<br>File: contracts/GeVault.sol
```
 
Line: 141          require(t0 == token0 && t1 == token1, "GEV: Invalid TR")
```
File: contracts/GeVault.sol
```
 
Line: 170          require(t0 == token0 && t1 == token1, "GEV: Invalid TR")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L120)


- File: contracts/GeVault.sol
```
 
Line: 203          require(poolMatchesOracle(), "GEV: Oracle Error")
```
Has duplicate:<br>File: contracts/GeVault.sol
```
 
Line: 215          require(poolMatchesOracle(), "GEV: Oracle Error")
```
File: contracts/GeVault.sol
```
 
Line: 250          require(poolMatchesOracle(), "GEV: Oracle Error")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L203](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L203)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 69          require( address(lendingPool) == msg.sender, "OPM: Call Unallowed")
```
Has duplicate:<br>File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 91          require( address(lendingPool) == msg.sender, "OPM: Call Unallowed")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L69](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L69)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 393          require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" )
```
Has duplicate:<br>File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 420          require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" )
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L393](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L393)


- File: contracts/helper/LPOracle.sol
```
 
Line: 63          require(timeStamp > 0, "Round not complete")
```
Has duplicate:<br>File: contracts/helper/OracleConvert.sol
```
 
Line: 44          require(timeStamp > 0, "Round not complete")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L63](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L63)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 87          require(acceptPayable, "CannotReceiveETH")
```
Has duplicate:<br>File: contracts/helper/V3Proxy.sol
```
 
Line: 91          require(acceptPayable, "CannotReceiveETH")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L87](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L87)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 99          require(path.length == 2, "Direct swap only")
```
Has duplicate:<br>File: contracts/helper/V3Proxy.sol
```
 
Line: 106          require(path.length == 2, "Direct swap only")
```
File: contracts/helper/V3Proxy.sol
```
 
Line: 113          require(path.length == 2, "Direct swap only")
```
File: contracts/helper/V3Proxy.sol
```
 
Line: 125          require(path.length == 2, "Direct swap only")
```
File: contracts/helper/V3Proxy.sol
```
 
Line: 138          require(path.length == 2, "Direct swap only")
```
File: contracts/helper/V3Proxy.sol
```
 
Line: 148          require(path.length == 2, "Direct swap only")
```
File: contracts/helper/V3Proxy.sol
```
 
Line: 161          require(path.length == 2, "Direct swap only")
```
File: contracts/helper/V3Proxy.sol
```
 
Line: 179          require(path.length == 2, "Direct swap only")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L99](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L99)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 139          require(path[0] == ROUTER.WETH9(), "Invalid path")
```
Has duplicate:<br>File: contracts/helper/V3Proxy.sol
```
 
Line: 149          require(path[0] == ROUTER.WETH9(), "Invalid path")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L139](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L139)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 162          require(path[1] == ROUTER.WETH9(), "Invalid path")
```
Has duplicate:<br>File: contracts/helper/V3Proxy.sol
```
 
Line: 180          require(path[1] == ROUTER.WETH9(), "Invalid path")
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L162](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L162)


</details>

# 


## Lack of Explicit Visibility Declaration in State Variables
- Severity: Non-Critical
- Confidence: High

### Description
This detector identifies state variables that are missing explicit visibility declarations. By default, without explicit declaration, the visibility of such variables is set to 'internal'. While this does not necessarily lead to vulnerabilities, explicitly declaring the visibility of each state variable can improve code readability and maintainability. Therefore, it is recommended to always specify the visibility, even when it's the default 'internal'.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/GeVault.sol
```
 
Line: 41          RangeManager rangeManager
```
State variable lacks explicit visibility.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L41](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L41)


- File: contracts/TokenisableRange.sol
```
 
Line: 45          string _name
```
State variable lacks explicit visibility.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L45](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L45)


- File: contracts/TokenisableRange.sol
```
 
Line: 46          string _symbol
```
State variable lacks explicit visibility.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L46](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L46)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 65          bool acceptPayable
```
State variable lacks explicit visibility.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L65](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L65)


</details>

# 



## Inconsistent usage of require/error
- Severity: Non-Critical
- Confidence: High

### Description
Some parts of the codebase use require statements, while others use custom errors. Consider refactoring the code to use the same approach: the following findings represent the minority of require vs error, and they show one occurrence in each contract, for brevity.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- File: contracts/RangeManager.sol
```
 
Line: 25          contract RangeManager is ReentrancyGuard, Ownable 
```
File: contracts/RangeManager.sol
```
 
Line: 66          revert("Range overlap")
```
The contract `RangeManager` uses both `require` and custom `revert` for error handling.

There are 5 `require` statement(s) and 1 custom `revert` statement(s).

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L25-L215](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L25-L215)


</details>

# 


## Token contract should have a blacklist function or modifier
- Severity: Non-Critical
- Confidence: Medium

### Description
NFT thefts have increased recently, so with the addition of stolen NFTs to the platform, NFTs can be converted into liquidity. To prevent this, we recommend adding a blacklist function.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/GeVault.sol
```
 
Line: 27          contract GeVault is ERC20, Ownable, ReentrancyGuard 
```
Token contract does not contain a blacklist. Consider adding one for increased security.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L27-L466](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L27-L466)


- File: contracts/TokenisableRange.sol
```
 
Line: 18          contract TokenisableRange is ERC20("", ""), ReentrancyGuard 
```
Token contract does not contain a blacklist. Consider adding one for increased security.
[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L18-L384](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L18-L384)


</details>

# 


## Variable names too similar
- Severity: Non-Critical
- Confidence: Medium

### Description
Detect variables with names that are too similar.

<details>

<summary>
There are 41 instances of this issue:

</summary>

###
- Variable File: interfaces/IUniswapV2Router01.sol
```
 
Line: 13          uint256 amountADesired
```
 is too similar to File: interfaces/IUniswapV2Router01.sol
```
 
Line: 14          uint256 amountBDesired
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV2Router01.sol#L13](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV2Router01.sol#L13)


- Variable File: interfaces/IUniswapV3SwapCallback.sol
```
 
Line: 17          int256 amount0Delta
```
 is too similar to File: interfaces/IUniswapV3SwapCallback.sol
```
 
Line: 18          int256 amount1Delta
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV3SwapCallback.sol#L17](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV3SwapCallback.sol#L17)


- Variable File: contracts/GeVault.sol
```
 
Line: 339          uint availToken0 = token0.balanceOf(address(this))
```
 is too similar to File: contracts/GeVault.sol
```
 
Line: 340          uint availToken1 = token1.balanceOf(address(this))
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L339](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L339)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 521          uint priceAssetA = oracle.getAssetPrice(assetA)
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 522          uint priceAssetB = oracle.getAssetPrice(assetB)
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L521](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L521)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 336          uint token0Amount
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 270          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L336](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L336)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 270          uint token0Amount
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 336          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L270](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L270)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 270          uint token0Amount
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 360          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L270](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L270)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 336          uint token0Amount
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 336          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L336](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L336)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 270          uint token0Amount
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 270          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L270](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L270)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 336          uint token0Amount
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 360          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L336](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L336)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 359          uint token0Amount
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 270          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L359](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L359)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 359          uint token0Amount
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 336          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L359](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L359)


- Variable File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 359          uint token0Amount
```
 is too similar to File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 360          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L359](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L359)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 269          uint256 TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token))
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 270          uint256 TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token))
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L269](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L269)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 352          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 270          uint256 TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token))
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 343          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 270          uint256 TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token))
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L343](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L343)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 333          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 270          uint256 TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token))
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 269          uint256 TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token))
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 352          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L269](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L269)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 352          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 352          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 343          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 352          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L343](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L343)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 333          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 352          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 343          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 343          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L343](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L343)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 343          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 333          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L343](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L343)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 352          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 333          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 269          uint256 TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token))
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 343          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L269](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L269)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 333          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 333          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 269          uint256 TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token))
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 333          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L269](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L269)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 352          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 343          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 333          uint TOKEN0_PRICE
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 343          uint TOKEN1_PRICE
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L333)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 240          uint256 token0Amount
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 368          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L240](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L240)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 368          uint token0Amount
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 368          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L368](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L368)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 240          uint256 token0Amount
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 377          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L240](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L240)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 240          uint256 token0Amount
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 240          uint256 token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L240](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L240)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 377          uint token0Amount
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 377          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L377](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L377)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 368          uint token0Amount
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 377          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L368](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L368)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 368          uint token0Amount
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 240          uint256 token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L368](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L368)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 377          uint token0Amount
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 368          uint token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L377](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L377)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 377          uint token0Amount
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 240          uint256 token1Amount
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L377](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L377)


- Variable File: contracts/TokenisableRange.sol
```
 
Line: 204          uint token0Price = ORACLE.getAssetPrice(address(TOKEN0.token))
```
 is too similar to File: contracts/TokenisableRange.sol
```
 
Line: 205          uint token1Price = ORACLE.getAssetPrice(address(TOKEN1.token))
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L204](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L204)


- Variable File: interfaces/INonfungiblePositionManager.sol
```
 
Line: 73          uint256 feeGrowthInside0LastX128
```
 is too similar to File: interfaces/INonfungiblePositionManager.sol
```
 
Line: 74          uint256 feeGrowthInside1LastX128
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L73](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L73)


- Variable File: interfaces/INonfungiblePositionManager.sol
```
 
Line: 75          uint128 tokensOwed0
```
 is too similar to File: interfaces/INonfungiblePositionManager.sol
```
 
Line: 76          uint128 tokensOwed1
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L75](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L75)


</details>

# 


## Single Source of Constants Ensuring Consistency and Integrity
- Severity: Non-Critical
- Confidence: High

### Description
It suggests that constants should be defined in only one contract to prevent values from becoming out of sync when updates are made in different locations.

Consider consolidating constant definitions in a single contract or using a centralized approach, such as creating an internal constant in a library. This ensures that all references to the constant retrieve the same value, eliminating inconsistencies and the potential for errors caused by mismatched values.

Additionally, if a variable is used as a local cache of another contract's value, it is recommended to make the cache variable internal or private. By doing so, external users are required to query the contract with the source of truth, ensuring that callers always obtain the most up-to-date and accurate information.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88) is redefines:
-    File: contracts/RangeManager.sol
```
 
Line: 36          INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88)
```

-    File: contracts/TokenisableRange.sol
```
 
Line: 58          INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88)
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36)


</details>

# 



## Missing inheritance
- Severity: Non-Critical
- Confidence: High

### Description
This detector identifies instances where a contract defines functions that match an interface's signature, but does not explicitly inherit from the interface. Explicitly declaring inheritance relationships can improve code readability and maintainability, and ensures that the contract adheres to the defined interface.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
- File: contracts/helper/LPOracle.sol
```
 
Line: 17          contract LPOracle 
```
 should inherit from File: contracts/helper/LPOracle.sol
```
 
Line: 13          interface IERC20 
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L17-L105](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L17-L105)


- File: contracts/helper/OracleConvert.sol
```
 
Line: 15          contract OracleConvert 
```
 should inherit from File: contracts/helper/LPOracle.sol
```
 
Line: 13          interface IERC20 
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L15-L81](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L15-L81)


- File: contracts/helper/V3Proxy.sol
```
 
Line: 60          contract V3Proxy is ReentrancyGuard, Ownable 
```
 should inherit from File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 7          interface  AmountsRouter 
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L60-L196](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L60-L196)


</details>

# 


## Max Value Optimization Replacing 2^n - 1 with type(uint n).max
- Severity: Non-Critical
- Confidence: High

### Description
In Solidity, 2**<n> - 1 is commonly used to represent the maximum value for an unsigned integer of n bits. However, this expression can be replaced with type(uint<n>).max, which provides a more explicit and self-explanatory representation of the maximum value.If the -1 adjustment is not present, you can still use type(uint<n>).max and add 1 to obtain the correct maximum value.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
- File: contracts/GeVault.sol
```
 
Line: 374          priceX8 = priceX8 * uint(sqrtPriceX96 / 2 ** 12) ** 2 * 1e8 / 2**168
```
 should use max() instead

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L374](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L374)


- File: contracts/TokenisableRange.sol
```
 
Line: 99          int24 _upperTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(startX10) * 10 ** TOKEN0.decimals) ) ) )
```
 should use max() instead

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L99](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L99)


- File: contracts/TokenisableRange.sol
```
 
Line: 100          int24 _lowerTick = TickMath.getTickAtSqrtRatio( uint160( 2**48 * sqrt( (2 ** 96 * (10 ** TOKEN1.decimals)) * 1e10 / (uint256(endX10  ) * 10 ** TOKEN0.decimals) ) ) )
```
 should use max() instead

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L100](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L100)


- File: contracts/TokenisableRange.sol
```
 
Line: 338          (amt0, amt1) = LiquidityAmounts.getAmountsForLiquidity( uint160( sqrt( (2 ** 192 * ((TOKEN0_PRICE * 10 ** TOKEN1.decimals) / TOKEN1_PRICE)) / ( 10 ** TOKEN0.decimals ) ) ), TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  liquidity)
```
 should use max() instead

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L338](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L338)


</details>

# 



## Name reused
- Severity: Non-Critical
- Confidence: High

### Description
If a codebase has two contracts the similar names, the compilation artifacts
will not contain one of the contracts with the duplicate name.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
- ISwapRouter is re-used:
	- File: contracts/helper/V3Proxy.sol
```
 
Line: 9          interface ISwapRouter 
```

	- File: interfaces/ISwapRouter.sol
```
 
Line: 9          interface ISwapRouter is IUniswapV3SwapCallback 
```


[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L9-L35](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L9-L35)


</details>

# 



## Override function arguments that are unused should have the variable name removed or commented out
- Severity: Non-Critical
- Confidence: High

### Description
Unused arguments in overridden functions can lead to compiler warnings andcan make the code less readable. It is a good practice to remove or comment outunused arguments in these cases.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 41          uint256[] calldata premiums
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L41](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L41)


- File: contracts/PositionManager/OptionsPositionManager.sol
```
 
Line: 42          address initiator
```

[https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L42](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L42)


</details>

# 