
# LOW RISK

| Number | Issue | instances |
|--------|-------|-----------|
|[L-01]| Array lengths not checked  | 5 |
|[L-02]| Burn functions should be protected with a modifier  | 2 |
|[L-03]| onlyOwner functions not accessible if owner renounces ownership  | 14 |
|[L-04]| Constructor missing zero address check  | 3 |
|[L-05]| Use safeApprove instead of approve  | 1 |
|[L-06]| approve can revert if the current approval is not zero  | 1 |


## [L-01] Array lengths not checked

If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array


```solidity
file:   contracts/PositionManager/OptionsPositionManager.sol

38     function executeOperation(
    address[] calldata assets,
    uint256[] calldata amounts,
    uint256[] calldata premiums,
    address initiator,
    bytes calldata params
  ) override external returns


61    function executeBuyOptions(
    uint poolId,
    address[] calldata assets,
    uint256[] calldata amounts,
    address user,
    address[] memory sourceSwap
  ) internal

83     function executeLiquidation(
    uint poolId,
    address[] calldata assets,
    uint256[] calldata amounts,
    address user,
    address collateral
  ) internal

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L38-L44

```solidity
file:  contracts/interfaces/IAaveIncentivesController.sol
 
98      function claimRewards(
    address[] calldata assets,
    uint256 amount,
    address to
  ) external returns (uint256);

112    function claimRewardsOnBehalf(
    address[] calldata assets,
    uint256 amount,
    address user,
    address to
  ) external returns (uint256);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/interfaces/IAaveIncentivesController.sol#L98-L102  

## [L-02] Burn functions should be protected with a modifier

```solidity
file:  contracts/interfaces/IAToken.sol

61     function burn(

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/interfaces/IAToken.sol#L61

```solidity
file: contracts/interfaces/INonfungiblePositionManager.sol

179    function burn(uint256 tokenId) external payable;

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/interfaces/INonfungiblePositionManager.sol#L179

## [L-03] onlyOwner functions not accessible if owner renounces ownership

The owner is able to perform certain privileged activities, but it's possible to set the owner to address(0). This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, therefore limiting any functionality that needs authority.

```solidity
file:  contracts/GeVault.sol

101  function setEnabled(bool _isEnabled) public onlyOwner { 

018  function setTreasury(address newTreasury) public onlyOwner { 

116  function pushTick(address tr) public onlyOwner {

137  function shiftTick(address tr) public onlyOwner {

167  function modifyTick(address tr, uint index) public onlyOwner {

183  function setBaseFee(uint newBaseFeeX4) public onlyOwner {

191  function setTvlCap(uint newTvlCap) public onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101

```solidity
file:  contracts/RangeManager.sol

75   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {

95   function initRange(address tr, uint amount0, uint amount1) external onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L75

```solidity
file: contracts/RoeRouter.sol

48   function deprecatePool(uint poolId) public onlyOwner {

59    function addPool(
    address lendingPoolAddressProvider, 
    address token0, 
    address token1, 
    address ammRouter
  ) 
    public onlyOwner 

83   function setTreasury(address newTreasury) public onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L48

```solidity
file: contracts/helper/FixedOracle.sol

24     function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L24

```solidity
file:  contracts/helper/V3Proxy.sol

94    function emergencyWithdraw(ERC20 token) onlyOwner external {  

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L94

## [L-04] Constructor missing zero address check

It is important to ensure that the constructor does not allow zero address to be set. This is a common mistake that can lead to loss of funds or redeployment of the contract.

```solidity
file: contracts/helper/FixedOracle.sol

15   constructor(int256 _hardcodedPrice) {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol#L15

```solidity
file:  contracts/helper/V3Proxy.sol

75    constructor(ISwapRouter _router, IQuoter _quoter, uint24 _fee) {

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L75


```solidity
file:  contracts/PositionManager/OptionsPositionManager.sol

25      constructor (address roerouter) PositionManager(roerouter) {}

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L25


## [L-05] Use safeApprove instead of approve

The approve function in ERC20 tokens is not safe to use. It is possible for an attacker to front-run the transaction and steal the tokens. Use safeApprove instead.

```solidity
file:  contracts/RangeManager.sol

165    tr.approve(address(LENDING_POOL), lpAmt);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L165


## [L-06] approve can revert if the current approval is not zero

Some tokens like USDT check for the current approval, and they revert if it's not zero. While Tether is known to do this, it applies to other tokens as well, which are trying to protect against this attack vector.

```solidity
file:  contracts/RangeManager.sol

165    tr.approve(address(LENDING_POOL), lpAmt);

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L165

### Recommended Mitigation Steps

Consider always calling token.safeApprove(addr, 0) before changing the approval, or better yet, use safeIncreaseAllowance/safeDecreaseAllowance.

