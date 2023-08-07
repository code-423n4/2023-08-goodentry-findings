# Analysis - Good Entry 

## Approach taken in evaluating the codebase

### Steps:

- Read all documents with very fast forward way
- Then read the Readme.md file understood following things 

   - The test coverage is only 85%
   - Protocol uses Aave LP and Uniswap v3 positions implementations 
   - Protocol uses EVM-compatible side-chain
   - Protocol uses Chainlink oracle 
   - AMM, ERC-20 Token
   -  list specific ERC20 that your protocol is anticipated to interact with USDC.e (bridged USDC), wBTC, ARB, GMX
   -  Smart contracts deployed Arbitrum 
   -  EIP's used TokenizableRange.sol and GeVault.sol are ERC20 compliant
- GoodEntry is a perpetual options trading platform, or protected perps: user can trade perps with limited downside. It is built on top of Uniswap v3 and relies on single tick liquidity

  - Tokenized Uniswap v3 positions (TR), with a manager, oracle, and price manipulation resistance
  - ezVaults, holding TRs, and rebalancing the underlying liquidity as needed
  - A lending pool, forked from Aave v2 (out of audit scope)
  - A router that whitelists allowed addresses (such as allowed swap pools)

- Then clone the repo and setup test environment to make sure all written tests are working correct
- Then Jump into the codebase analysis. In first iteration just identified the important GAS and QA related findings. 
#### GAS

1. Using ``calldata`` instead of memory for read-only arguments in external functions saves gas
2. State variables can be packed into fewer storage slots
3. Structs can be packed into fewer storage slots
4. Using bools for storage incurs overhead
5. Functions guaranteed to revert when called by normal users can be marked payable
6. Multiple accesses of a mapping/array should use a local variable cache
7. State variable should be cached inside the loop
8. IF’s/require() statements that check input arguments should be at the top of the function

### QA

1. Array is ``push()ed`` but not ``pop()ed``
2. Missing ``contract-existence`` checks before low-level calls
3. External call ``recipient`` may consume all transaction gas
4. Allowed fees/rates should be capped by smart contracts
5. Consider implementing two-step procedure for updating protocol addresses
6. ``safeApprove()`` is deprecated
7. Calls to withdraw() will revert when totalSupply() returns zero
8. Some tokens may revert when zero value transfers are made
9. Hardcoded ``treasuryFee`` may cause problem in future 
10. Remove deprecated variables to avoid confusion 

- Then deep dig to the documents thoroughly and understood protocols flow
-Deep dive into code base with line by line analysis. The weak and vulnerable areas are marked with @audit tags
- Then analyzing each and every @audit tags more deep. Then performed all possible unit and fuzzing tests to ensured the functions are working indented way.
- Then i reported final reports as per C4 Formats 
#### M/H Risks

1. Call function without checking status may lead to fund loss
2. Chainlink's ``latestRoundData`` return stale or incorrect result
   

## Codebase quality analysis and Possible Risks Comments

[OptionsPositionManager.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol)

#### Code
1. In the executeOperation function, you can avoid the need to decode the params twice by using a local variable for the decoded mode value
2. Choose more descriptive variable names to improve code readability. For instance, replace amtA and amtB with more meaningful names that convey their purpose
3. The executeBuyOptions and executeLiquidation functions can be quite complex. Consider breaking down their functionality into smaller helper functions with well-defined purposes to improve code maintainability
4. Reduce nesting levels in functions such as closeDebt to enhance readability. Nested statements can become hard to follow, and breaking them down can help mitigate this issue
5. For repetitive actions like swapping tokens or interacting with the lending pool, consider centralizing common functionality into utility functions to reduce code duplication and promote consistency
6. Explicitly specify the visibility modifiers for functions to ensure clarity and to prevent accidental misuse
7. Replace hardcoded values like 1e18 or 1e8 with named constants or variables with explanatory names.
8. Provide detailed error messages in require statements to give better insight into why a condition failed and how to address the issue

#### Risks
- Ensure that only authorized users or contracts can call critical functions like ``liquidation``
- The code may be susceptible to reentrancy attacks if external calls are made before ensuring the current execution context is secure. This could potentially allow malicious contracts to call back into your contract and manipulate its state.

[OptionsPositionManager.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol)

#### Code

1. Consider using the view or pure keywords for functions that don't modify state. This can help save gas when users query the contract
2. Use internal instead of public for returnExpectedBalanceWithoutFees:
Since returnExpectedBalanceWithoutFees is intended to be used only within the contract, you can make it an internal function to prevent unnecessary external visibility
3. In the latestAnswer function, you can store the asset prices obtained from the Oracle in variables to avoid making duplicate calls
4. In the getTokenAmounts function, consider reusing the getTokenAmountsExcludingFees function instead of duplicating its logic
5. The code contains multiple conditional statements, calculations, and interactions with external contracts. This complexity could make the code harder to understand, debug, and maintain
6. Some parts of the code lack explanatory comments, which could make it challenging for other developers to understand the intentions and assumptions behind specific calculations.
7. The naming conventions for functions and variables are inconsistent, which could lead to confusion and misunderstandings among developers

### Risks

- The contract relies on external price data for asset tokens (TOKEN0_PRICE and TOKEN1_PRICE) from an oracle. Inaccurate or manipulated price data could lead to incorrect fee calculations, affecting user balances and overall contract behavior
- The calculation of feeLiquidity and the subsequent distribution of fees may have precision and rounding issues. This could impact fee accuracy and user expectations when interacting with the contract
- The nonReentrant modifier is used, but it's important to ensure that external contract calls, such as POS_MGR.increaseLiquidity and POS_MGR.decreaseLiquidity, are placed after state changes to prevent reentrancy vulnerabilities
- The contract relies on external contracts (e.g., Uniswap V3 contracts) for key operations. Changes or upgrades to these contracts could impact the behavior and reliability of this contract
- External contract interactions, such as with Uniswap V3, could expose the contract to front-running attacks, where malicious actors manipulate transactions to their advantage
- While the contract checks minimum withdrawal amounts (amount0Min and amount1Min), it doesn't validate that these values are within reasonable ranges or handle potential edge cases
- There's no mechanism to handle cases where liquidity is removed from the contract, potentially resulting in stranded liquidity or unexpected contract behavior

[RangeManager.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol)

#### Code

1.  Instead of using if statements followed by revert, you can use the require statement directly, which provides a more concise and readable way to check conditions and revert if they're not met
2.  If the ranges have different states (e.g., active, closed), consider using an enum to define these states and track them in the Step struct
3.  Consider using more compact data types when possible to reduce storage costs. For example, if the range IDs or step IDs won't exceed a certain limit, you can use smaller integer types like uint32 instead of uint256
4.  Consider using the try / catch pattern for handling external contract calls to gracefully handle failures and avoid stopping contract execution completely

#### Risks

- The contract interacts with tokens using safeIncreaseAllowance. Ensure that you manage token approvals properly to prevent unauthorized transfers and potential attacks
- The checkNewRange function checks for overlaps between new ranges and existing ones. If there's a bug in this logic, it could lead to range overlaps, potentially causing unintended behavior
- The contract interacts with various external contracts, including the Aave lending pool, Uniswap, and more. Be aware that external contracts can change behavior, be upgraded, or become malicious, which might impact your contract's operation
- Transactions and state changes on the blockchain are visible to the public. If there are economically significant operations that can be front-run, malicious actors might take advantage of them
- Depending on the interaction with the Aave lending pool and Uniswap, there might be risks associated with the security and liquidity of these pools

[PositionManager.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol)

#### Code

1. Some functions lack visibility specifiers (public, internal, external, private), which can lead to confusion and unexpected behavior. Always specify the intended visibility of functions
2. Unused import statements, such as IUniswapV2Pair, IUniswapV2Factory, etc., suggest that the contract is importing unnecessary interfaces, which can increase deployment gas costs.
3. State variables like LENDING_POOL and ADDRESSES_PROVIDER are declared but not initialized in the constructor, which can lead to unexpected behaviour
4. Constructor parameters are not checked for validity, such as ensuring that the roerouter address is not address(0)
5. Arrays like tokenisedRanges and tokenisedTicker are maintained even though their elements might be removed. This can lead to inefficient use of storage and increase gas costs

#### Risks

- There are instances where variables are declared but not used. This could indicate unnecessary complexity or potential issues with logic
- The usage of type(uint256).max for setting allowances can be risky, as it may exceed the available gas limit in a transaction. It's better to set allowances to a reasonable value
- The cleanup function doesn't handle the case where repay or deposit calls fail, which can result in partial or failed cleanup operations.
- The direct use of .balanceOf() on an ERC20 token can cause issues if the token doesn't implement the ERC20 standard properly. It's safer to use the SafeERC20 library
- In the validateValuesAgainstOracle function, the check for value comparisons might not be accurate due to precision issues with decimals
- In the checkNewRange function, when detecting overlapping ranges, the contract immediately reverts. This doesn't provide any way to handle overlapping ranges, potentially leading to failed transactions and user confusion
- In the cleanup function, you're calling the repay and deposit functions of the lending pool without checking their return values. If these operations fail, the contract won't handle it, potentially leading to funds getting stuck
- Ensure that the arithmetic operations involving token amounts and prices are performed with correct precision to prevent unintended consequences due to rounding errors


## Gas Optimization

This analysis report is meant to accompany my Gas Optimizations Report submission. All insights will be given through the perspective of optimizing the codebase

1. By utilizing calldata instead of memory arrays, the contract avoids the gas-intensive abi.decode() loop for array copying. This saves gas by bypassing the need for individual index copying.
2. Optimally packing state variables within 32-byte storage slots efficiently uses storage space. Fewer slots lead to less SLOAD usage, resulting in substantial gas savings
3. Efficiently packing structs into fewer storage slots avoids extra SLOAD operations. Using smaller types, like uint96, further reduces storage costs and saves gas
4. Replacing bools with uint256 (1 or 2) bypasses extra SLOADs, while marking guaranteed-to-revert functions as payable reduces gas costs by avoiding unnecessary checks
5. Caching mapping and array accesses in local variables saves gas by eliminating recalculations of key hashes and offsets for each access in the loop
6. Caching state variables inside loops reduces gas costs by minimizing storage slot accesses, improving execution efficiency.
7. Moving constant checks and input argument validations to the top of functions minimizes gas costs for invalid input by failing quickly with low gas costs.
8. When functions are equipped with a modifier like onlyOwner, they automatically revert if regular users attempt to make calls. Marking these functions as payable lowers gas costs for legitimate callers since the compiler omits payment checks. By avoiding extra opcodes like CALLVALUE, DUP1, ISZERO, PUSH2, JUMPI, PUSH1, DUP1, REVERT, JUMPDEST, and POP, each call saves around 21 gas on average. Additionally, this optimization reduces deployment costs

These optimizations, backed by opcode explanations, enhance the contract's gas efficiency and overall performance


## Centralization risks

The "onlyowner" keyword in smart contracts allows a single address to have complete control over the contract. This can lead to centralization risks, as the single owner could potentially abuse their power

The single owner could:

1. Freeze the funds of other users.
2. Change the rules of the contract.
3. Terminate the contract.

```solidity
101:   function setEnabled(bool _isEnabled) public onlyOwner  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101-L101

```solidity
108:   function setTreasury(address newTreasury) public onlyOwner  
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L108-L108

```solidity
116:   function pushTick(address tr) public onlyOwner  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L116-L116

```solidity
137:   function shiftTick(address tr) public onlyOwner  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L137-L137

```solidity
167:   function modifyTick(address tr, uint index) public onlyOwner  
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L167-L167

```solidity
183:   function setBaseFee(uint newBaseFeeX4) public onlyOwner  
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L183-L183


```solidity
191:   function setTvlCap(uint newTvlCap) public onlyOwner  
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L191-L191

```solidity
94:     function emergencyWithdraw(ERC20 token) onlyOwner external  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L94-L94


```solidity
75:   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner  
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75-L75

```solidity
95:   function initRange(address tr, uint amount0, uint amount1) external onlyOwner 
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L95-L95

```solidity
48:   function deprecatePool(uint poolId) public onlyOwner 
```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L48-L48

```solidity

59:   function addPool(
60:     address lendingPoolAddressProvider, 
61:     address token0, 
62:     address token1, 
63:     address ammRouter
64:   ) 
65:     public onlyOwner 
66:     returns (uint poolId)
67:   
```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L65-L65

```solidity
24:     function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner 

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24-L24

## Time Spend
10 hours 

### Time spent:
10 hours