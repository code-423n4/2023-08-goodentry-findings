## [L-01] Critical Address Changes Should Use Two-step Procedure
The critical procedures should be a two-step process.
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
**Recommended Mitigation Steps**
Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two-step procedure on the critical functions.

# Non-Critical

## [N-01] Generate perfect code headers every time
Description:
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```


## [N-02] Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

## [N-03] Add a timelock to critical functions
It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

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

## [N-4] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L546C4-L547C92

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L266

## [N-5] remove the spaces for better readability
```
function name()     public view virtual override returns (string memory) { return _name; }
  /// @notice Get the symbol of this contract token
  function symbol()   public view virtual override returns (string memory) { return _symbol; }
```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L125C3-L127C95

## [N-6] Pragma version^0.8.19 version too recent to be trusted.
https://github.com/ethereum/solidity/blob/develop/Changelog.md
0.8.19 (2023-02-22)
[Moonwell](:/d21d89f7599840ab8d78de7f85d0eed6)0.8.17 (2022-09-08)
0.8.16 (2022-08-08)
0.8.15 (2022-06-15)
0.8.10 (2021-11-09)

Unexpected bugs can be reported in recent versions;

- Risks related to recent releases
- Risks of complex code generation changes
- Risks of new language features
- Risks of known bugs
Use a non-legacy and more battle-tested version
Use 0.8.10

##  [S-01] You can explain the operation of critical functions in NatSpec with an infographic.