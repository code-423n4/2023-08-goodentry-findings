|ID|Title|Category|Severity|Instances|
|-|:-|:-:|:-:|:-:|
| [[1](#nc-1--use-modifier-instead-of-requireif-for-special-msgsender)] | Use modifier instead of require/if for special `msg.sender` | Control Flow | Informational | 3 |
| [[2](#nc-2--lack-of-brace-spacing)] | Lack Of Brace Spacing | Coding Style | Informational | 37 |
| [[3](#nc-3--approve-is-fine-as-it-cannot-be-front-run)] | Approve is fine as it cannot be front-run | Volatile Code | Informational | 12 |
| [[4](#nc-4--importing-ierc20-is-sufficient-for-erc20-transactions)] | Importing IERC20 is sufficient for ERC20 transactions | Volatile Code | Informational | 5 |
| [[5](#nc-5--unused-state-variables)] | Unused State variables | Gas Optimization | Informational | 7 |
| [[6](#nc-6--file-is-miss-spdx-license-identifier)] | File is miss SPDX-License-Identifier | Coding Style | Informational | 1 |
| [[7](#nc-7--events-may-be-emitted-out-of-order-due-to-reentranc)] | Events may be emitted out of order due to reentranc | Logical Issue | Informational | 4 |
| [[8](#nc-8--emitting-storage-values-instead-of-the-memory-one)] | Emitting storage values instead of the memory one. | Gas Optimization | Informational | 1 |

## **NC-1** | Use modifier instead of require/if for special `msg.sender`

### **Description** :-

If functions are only allowed to be called by the special actor, modifier should be used instead of checking with require statement, if actor is the msg.sender calling the function.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-goodentry/contracts/GeVault.sol
```
```solidity
463:if(msg.sender != address(WETH)
```
#### **Context**:-
[GeVault.sol#L463](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L463)
```solidity
File:2023-08-goodentry/contracts/TokenisableRange.sol
```
```solidity
136:require(msg.sender == creator, "Unallowed call")
```
#### **Context**:-
[TokenisableRange.sol#L136](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L136)
```solidity
File:2023-08-goodentry/contracts/helper/FixedOracle.sol
```
```solidity
11:require(msg.sender == owner, "Only the owner can call this function.")
```
#### **Context**:-
[FixedOracle.sol#L11](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L11)

## **NC-2** | Lack Of Brace Spacing

### **Description** :-

There are many instances of non-spaced function / conditional braces. Consider adding a space between the function / conditional closing parenthesis and brace.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-goodentry/contracts/PositionManager/OptionsPositionManager.sol
```
```solidity
47:){
```
#### **Context**:-
[OptionsPositionManager.sol#L47](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L47)
```solidity
71:){
```
#### **Context**:-
[OptionsPositionManager.sol#L71](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L71)
```solidity
93:){
```
#### **Context**:-
[OptionsPositionManager.sol#L93](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L93)
```solidity
136:){
```
#### **Context**:-
[OptionsPositionManager.sol#L136](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L136)
```solidity
172:){
```
#### **Context**:-
[OptionsPositionManager.sol#L172](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L172)
```solidity
203:){
```
#### **Context**:-
[OptionsPositionManager.sol#L203](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L203)
```solidity
275:){
```
#### **Context**:-
[OptionsPositionManager.sol#L275](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L275)
```solidity
295:){
```
#### **Context**:-
[OptionsPositionManager.sol#L295](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L295)
```solidity
300:){
```
#### **Context**:-
[OptionsPositionManager.sol#L300](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L300)
```solidity
473:){
```
#### **Context**:-
[OptionsPositionManager.sol#L473](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L473)
```solidity
544:){
```
#### **Context**:-
[OptionsPositionManager.sol#L544](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L544)
```solidity
File:2023-08-goodentry/contracts/GeVault.sol
```
```solidity
153:){
```
#### **Context**:-
[GeVault.sol#L153](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L153)
```solidity
177:){
```
#### **Context**:-
[GeVault.sol#L177](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L177)
```solidity
230:){
```
#### **Context**:-
[GeVault.sol#L230](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L230)
```solidity
255:){
```
#### **Context**:-
[GeVault.sol#L255](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L255)
```solidity
298:){
```
#### **Context**:-
[GeVault.sol#L298](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L298)
```solidity
299:){
```
#### **Context**:-
[GeVault.sol#L299](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L299)
```solidity
314:){
```
#### **Context**:-
[GeVault.sol#L314](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L314)
```solidity
329:){
```
#### **Context**:-
[GeVault.sol#L329](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L329)
```solidity
346:){
```
#### **Context**:-
[GeVault.sol#L346](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L346)
```solidity
352:){
```
#### **Context**:-
[GeVault.sol#L352](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L352)
```solidity
356:){
```
#### **Context**:-
[GeVault.sol#L356](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L356)
```solidity
367:){
```
#### **Context**:-
[GeVault.sol#L367](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L367)
```solidity
392:){
```
#### **Context**:-
[GeVault.sol#L392](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L392)
```solidity
393:){
```
#### **Context**:-
[GeVault.sol#L393](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L393)
```solidity
404:){
```
#### **Context**:-
[GeVault.sol#L404](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L404)
```solidity
410:){
```
#### **Context**:-
[GeVault.sol#L410](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L410)
```solidity
429:){
```
#### **Context**:-
[GeVault.sol#L429](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L429)
```solidity
431:){
```
#### **Context**:-
[GeVault.sol#L431](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L431)
```solidity
File:2023-08-goodentry/contracts/TokenisableRange.sol
```
```solidity
237:){
```
#### **Context**:-
[TokenisableRange.sol#L237](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L237)
```solidity
268:){
```
#### **Context**:-
[TokenisableRange.sol#L268](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L268)
```solidity
368:){
```
#### **Context**:-
[TokenisableRange.sol#L368](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L368)
```solidity
377:){
```
#### **Context**:-
[TokenisableRange.sol#L377](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L377)
```solidity
File:2023-08-goodentry/contracts/PositionManager/PositionManager.sol
```
```solidity
96:){
```
#### **Context**:-
[PositionManager.sol#L96](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L96)
```solidity
125:){
```
#### **Context**:-
[PositionManager.sol#L125](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L125)
```solidity
File:2023-08-goodentry/contracts/helper/LPOracle.sol
```
```solidity
28:){
```
#### **Context**:-
[LPOracle.sol#L28](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol#L28)
```solidity
File:2023-08-goodentry/contracts/helper/OracleConvert.sol
```
```solidity
22:){
```
#### **Context**:-
[OracleConvert.sol#L22](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol#L22)


## **NC-3** | Approve is fine as it cannot be front-run

### **Description** :-



#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-goodentry/contracts/GeVault.sol
```
```solidity
386:.safeIncreaseAllowance(spender, type(uint256)
```
#### **Context**:-
[GeVault.sol#L386](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L386)
```solidity
File:2023-08-goodentry/contracts/TokenisableRange.sol
```
```solidity
140:.safeIncreaseAllowance(address(POS_MGR)
```
#### **Context**:-
[TokenisableRange.sol#L140](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L140)
```solidity
141:.safeIncreaseAllowance(address(POS_MGR)
```
#### **Context**:-
[TokenisableRange.sol#L141](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L141)
```solidity
191:.safeIncreaseAllowance(address(POS_MGR)
```
#### **Context**:-
[TokenisableRange.sol#L191](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L191)
```solidity
192:.safeIncreaseAllowance(address(POS_MGR)
```
#### **Context**:-
[TokenisableRange.sol#L192](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L192)
```solidity
249:.safeIncreaseAllowance(address(POS_MGR)
```
#### **Context**:-
[TokenisableRange.sol#L249](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L249)
```solidity
250:.safeIncreaseAllowance(address(POS_MGR)
```
#### **Context**:-
[TokenisableRange.sol#L250](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L250)
```solidity
File:2023-08-goodentry/contracts/RangeManager.sol
```
```solidity
97:.safeIncreaseAllowance(tr, amount0)
```
#### **Context**:-
[RangeManager.sol#L97](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L97)
```solidity
File:2023-08-goodentry/contracts/RangeManager.sol
```
```solidity
99:.safeIncreaseAllowance(tr, amount1)
```
#### **Context**:-
[RangeManager.sol#L99](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L99)
```solidity
File:2023-08-goodentry/contracts/RangeManager.sol
```
```solidity
156:.safeIncreaseAllowance(address(tr)
```
#### **Context**:-
[RangeManager.sol#L156](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L156)
```solidity
161:.safeIncreaseAllowance(address(tr)
```
#### **Context**:-
[RangeManager.sol#L161](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L161)
```solidity
File:2023-08-goodentry/contracts/PositionManager/PositionManager.sol
```
```solidity
81:.safeIncreaseAllowance(spender, type(uint256)
```
#### **Context**:-
[PositionManager.sol#L81](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L81)


## **NC-4** | Importing IERC20 is sufficient for ERC20 transactions

### **Description** :-

In order to run the functions of erc20, it is sufficient to import only IERC20.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:2023-08-goodentry/contracts/GeVault.sol
```
```solidity
5:import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol
```
#### **Context**:-
[GeVault.sol#L5](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L5)
```solidity
File:2023-08-goodentry/contracts/TokenisableRange.sol
```
```solidity
7:import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol
```
#### **Context**:-
[TokenisableRange.sol#L7](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L7)
```solidity
File:2023-08-goodentry/contracts/helper/V3Proxy.sol
```
```solidity
5:import "../openzeppelin-solidity/contracts/token/ERC20/ERC20.sol
```
#### **Context**:-
[V3Proxy.sol#L5](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L5)
```solidity
File:2023-08-goodentry/contracts/RangeManager.sol
```
```solidity
4:import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol
```
#### **Context**:-
[RangeManager.sol#L4](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L4)
```solidity
File:2023-08-goodentry/contracts/PositionManager/PositionManager.sol
```
```solidity
3:import "../openzeppelin-solidity/contracts/token/ERC20/ERC20.sol
```
#### **Context**:-
[PositionManager.sol#L3](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L3)

## **NC-5** | Unused State variables
### **Description** :-
Consider removing unused State variables or moving it to a higher level contract.

#### <ins>Proof Of Concept</ins>


```solidity
File:2023-08-goodentry/contracts/GeVault.sol
```

```solidity
41:  RangeManager rangeManager; 
```
#### **Context**:-
[GeVault.sol#L41](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L41)

```solidity
62:  uint public constant nearbyRanges = 2;
```
#### **Context**:-
[GeVault.sol#L62](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L62)

```solidity
File:2023-08-goodentry/contracts/TokenisableRange.sol
```

```solidity
54:  address public TREASURY_DEPRECATED = 0x22Cc3f665ba4C898226353B672c5123c58751692;
```
#### **Context**:-
[TokenisableRange.sol#L54](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L54)

```solidity
55:  uint public treasuryFee_deprecated = 20;
```
#### **Context**:-
[TokenisableRange.sol#L55](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L55)

```solidity
File:2023-08-goodentry/contracts/RangeManager.sol
```

```solidity
36:  INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);  
```
#### **Context**:-
[RangeManager.sol#L36](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L36)

```solidity
File:2023-08-goodentry/contracts/PositionManager/PositionManager.sol
```

```solidity
21:  ILendingPoolAddressesProvider public ADDRESSES_PROVIDER; // IFlashLoanReceiver  requirement
```
#### **Context**:-
[PositionManager.sol#L21](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L21)

```solidity
22:  ILendingPool public LENDING_POOL; // IFlashLoanReceiver  requirement
```
#### **Context**:-
[PositionManager.sol#L22](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L22)

## **NC-6** | File is miss SPDX-License-Identifier
### **Description** :-
contract is missing an SPDX identifier which correctly licenses the contract for open source development.

#### <ins>Proof Of Concept</ins>


```solidity
File:2023-08-goodentry/contracts/helper/FixedOracle.sol
```

```solidity
1:pragma solidity ^0.8.0;
```
#### **Context**:-
[FixedOracle.sol#L1](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L1)

## **NC-7** | Events may be emitted out of order due to reentranc
### **Description** :-
Ensure that events follow the best practice of check-effects-interaction, and are emitted before external calls

#### <ins>Proof Of Concept</ins>


```solidity
File:2023-08-goodentry/contracts/GeVault.sol
```

```solidity
240:    emit Withdraw(msg.sender, token, amount, liquidity);
```
#### **Context**:-
[GeVault.sol#L240](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L240)

```solidity
283:    emit Deposit(msg.sender, token, amount, liquidity);
```
#### **Context**:-
[GeVault.sol#L283](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L283)

```solidity
File:2023-08-goodentry/contracts/TokenisableRange.sol
```

```solidity
284:    emit Deposit(msg.sender, lpAmt);
```
#### **Context**:-
[TokenisableRange.sol#L284](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L284)

```solidity
326:    emit Withdraw(msg.sender, lp);
```
#### **Context**:-
[TokenisableRange.sol#L326](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L326)

## **NC-8** | Emitting storage values instead of the memory one.
### **Description** :-
Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. We can avoid unecessary SLOADs by caching storage values that were previously accessed and emitting those cached values.

#### <ins>Proof Of Concept</ins>


```solidity
File:2023-08-goodentry/contracts/GeVault.sol
```

```solidity
362:    emit Rebalance(tickIndex);
```
#### **Context**:-
[GeVault.sol#L362](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L362)