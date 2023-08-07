# LOW FINDINGS

##

## [L-1] Array is ``push()ed`` but not ``pop()ed``

### Impact
Array entries are added but are never removed. Consider whether this should be the case, or whether there should be a maximum, or whether old entries should be removed. Cases where there are specific potential problems will be flagged separately under a different issue.

### POC

```solidity
FILE: 2023-08-goodentry/contracts/GeVault.sol

121: if (ticks.length == 0) ticks.push(t);

130: ticks.push(TokenisableRange(tr));

152: ticks.push(ticks[ticks.length-1]);

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L121C3-L121C42

```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/RangeManager.sol

78: stepList.push( Step(startX10, endX10) );

80: tokenisedRanges.push( TokenisableRange(address(trbp)) );

82: tokenisedTicker.push( TokenisableRange(address(trbp)) );

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L78

```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/RoeRouter.sol

76: pools.push(RoePool(lendingPoolAddressProvider, token0, token1, ammRouter, false));

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RoeRouter.sol#L76


### Recommended Mitigation
Use ``pop()`` method to remove elements 


##

## [L-3] Missing ``contract-existence`` checks before low-level calls

### Impact
Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

### POC

```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/helper/V3Proxy.sol

174: payable(msg.sender).call{value: amountOut}("");

192: payable(msg.sender).call{value: amounts[1]}("");

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L174

### Recommended Mitigation
Add `` <address>.code.length > 0`` this check

##

## [L-4] External call ``recipient`` may consume all transaction gas

### Impact
There is no limit specified on the amount of gas used, so the recipient can use up all of the transaction's gas, causing it to revert. 

```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/helper/V3Proxy.sol

174: payable(msg.sender).call{value: amountOut}("");

192: payable(msg.sender).call{value: amounts[1]}("");

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L174

### Recommended Mitigation
Use ``addr.call{gas: <amount>}("")`` or this library instead.

##

## [L-5] Allowed fees/rates should be capped by smart contracts

### Impact
Fees/rates should be required to be below 100%, preferably at a much lower limit, to ensure users don't have to monitor the blockchain for changes prior to using the protocol

```
 /// @notice Max vault TVL with 8 decimals

```

### POC

```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/GeVault.sol

190: function setTvlCap(uint newTvlCap) public onlyOwner {
191:    tvlCap = newTvlCap;
192:    emit SetTvlCap(newTvlCap);
193:   }

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L191-L194

### Recommended Mitigation
Add ``sanity checks`` (Min,Max,>0) when assigning ``uint`` values 


##

## [L-6] Consider implementing two-step procedure for updating protocol addresses

### Impact
A copy-paste error or a typo may end up bricking protocol functionality, or sending tokens to an address with no known private key. Consider implementing a two-step procedure for updating protocol addresses, where the recipient is set as pending, and must 'accept' the assignment by making an affirmative call. A straight forward way of doing this would be to have the target contracts implement [EIP-165](https://eips.ethereum.org/EIPS/eip-165), and to have the 'set' functions ensure that the recipient is of the right interface type. 

### POC

```solidity
FILE: 2023-08-goodentry/contracts/RoeRouter.sol

 function setTreasury(address newTreasury) public onlyOwner {
    require(newTreasury != address(0x0), "Invalid address");
    treasury = newTreasury;
    emit UpdateTreasury(newTreasury);
  }

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RoeRouter.sol#L83-L87

```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/GeVault.sol

function setTreasury(address newTreasury) public onlyOwner { 
    treasury = newTreasury; 
    emit SetTreasury(newTreasury);
  }

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L108-L111

### Recommended Mitigation
Add ``two-step procedure`` when updating protocol addresses

##

## [L-7] ``safeApprove()`` is deprecated

### Impact
Deprecated in favor of safeIncreaseAllowance() and safeDecreaseAllowance(). If only setting the initial allowance to the value that means infinite, safeIncreaseAllowance() can be used instead. The function may currently work, but if a bug is found in this version of OpenZeppelin, and the version that you're forced to upgrade to no longer has this function, you'll encounter unnecessary delays in porting and testing replacement contracts.

### POC

```solidity
FILE: 2023-08-goodentry/contracts/helper/V3Proxy.sol

116: ogInAsset.safeApprove(address(ROUTER), amountIn);

120: ogInAsset.safeApprove(address(ROUTER), 0);

128: ogInAsset.safeApprove(address(ROUTER), amountInMax);

133: ogInAsset.safeApprove(address(ROUTER), 0);

165: ogInAsset.safeApprove(address(ROUTER), amountInMax);

169: ogInAsset.safeApprove(address(ROUTER), 0);

183: ogInAsset.safeApprove(address(ROUTER), amountIn);

187: ogInAsset.safeApprove(address(ROUTER), 0); 

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L116

### Recommended Mitigation
Use ``safeIncreaseAllowance()`` and ``safeDecreaseAllowance()``

##

## [L-8] Calls to withdraw() will revert when totalSupply() returns zero

### Impact
``totalSupply()`` being zero will result in a division by zero, causing the transaction to fail. The function should instead special-case this scenario, and avoid reverting.

### POC

```solidity
FILE: 2023-08-goodentry/contracts/GeVault.sol

221: uint valueX8 = vaultValueX8 * liquidity / totalSupply();

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L221

```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/TokenisableRange.sol

371: (token0Amount, token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick),  uint128 ( uint(liquidity) * amount / totalSupply() ) );

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L371

### Recommended Mitigation
``totalSupply()`` must be non zero 

##

[L-2] Some tokens may revert when zero value transfers are made

### Impact
In spite of the fact that EIP-20 states that zero-valued transfers must be accepted, some tokens, such as LEND will revert if this is attempted, which may cause transactions that involve other tokens (such as batch operations) to fully revert. Consider skipping the transfer if the amount is zero, which will also save gas.

### POC

```solidity
FILE: 2023-08-goodentry/contracts/GeVault.sol

227: ERC20(token).safeTransfer(treasury, fee);

235: ERC20(token).safeTransfer(msg.sender, bal);

267: ERC20(token).safeTransfer(treasury, fee);

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L267

### Recommended Mitigation
Check ``zero amount`` before call ``transfer`` function 


##

## [L-9] Hardcoded ``treasuryFee`` may cause problem in future 

### Impact
The fee may be difficult to change, which can make it difficult to adapt to changes in the market

### POC

```solidity
FILE: 2023-08-goodentry/contracts/TokenisableRange.sol

61: uint constant public treasuryFee = 20;

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L61

### Recommended Mitigation
Make sure ``treasuryFee`` easy to change when necessary 

##

## [L-10] Remove deprecated variables to avoid confusion 

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L53-L55

##

## [L-11] Add ``view`` when evert states are not changed

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L533

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L544

##

## [L-12] Pool deprecated ``status`` is not checked when ``accessing pools ``

### Impact

The deprecatePool function does not check the deprecated status of the pool before accessing it. This means that it is possible to call the getPool function with a deprecated pool id, and the function will return the pool's data, even though the pool is deprecated


### POC

```solidity
FILE: 2023-08-goodentry/contracts/PositionManager/PositionManager.sol

function getPoolAddresses(uint poolId) 
    internal view 
    returns( ILendingPool lp, IPriceOracle oracle, IUniswapV2Router01 router, address token0, address token1) 
  {
    (address lpap, address _token0, address _token1, address r, ) = ROEROUTER.pools(poolId);
    token0 = _token0;
    token1 = _token1;
    lp = ILendingPool(ILendingPoolAddressesProvider(lpap).getLendingPool());
    oracle = IPriceOracle(ILendingPoolAddressesProvider(lpap).getPriceOracle());
    router = IUniswapV2Router01(r);
  }

FILE: Breadcrumbs2023-08-goodentry/contracts/RoeRouter.sol

function deprecatePool(uint poolId) public onlyOwner {
    pools[poolId].isDeprecated = true;
    emit DeprecatePool(poolId);
  }

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/PositionManager.sol#L63C3-L73C4

### Recommended Mitigation
Add  check to ensure the pool is deprecated or not 

##

## [L-13] Shut off the renounceOwnership() function to avoid unexpected behavior 

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L4

##

## [L-14] Don’t use ``payable(address).call()``

### Impact
``payable(address).call()`` is a low-level method that can be used to send ether to a contract, but it has some limitations and risks as you've pointed out. One of the primary risks of using ``payable(address).call()`` is that it doesn't guarantee that the contract's payable function will be called successfully. This can lead to funds being lost or stuck in the contract

The contract does not have a payable callback The contract’s payable callback spends more than 2300 gas (which is only enough to emit something) The contract is called through a proxy which itself uses up the 2300 gas Use OpenZeppelin’s Address.sendValue() instead

```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/helper/V3Proxy.sol

174: payable(msg.sender).call{value: amountOut}("");

192: payable(msg.sender).call{value: amounts[1]}("");

```
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L174

### Recommended Mitigation
Use OpenZeppelin’s Address.sendValue()

##

## [L-15] Lack of control for liquidate() 

### Impact
Anyone can call and can initiate the liquidations . This may creates unintended consequences. 

### POC
```solidity
FILE: Breadcrumbs2023-08-goodentry/contracts/PositionManager/OptionsPositionManager.sol

function liquidate (
    uint poolId, 
    address user,
    address[] memory options, 
    uint[] memory amounts,
    address collateralAsset
  )
    external

``` 
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L189-L197

### Recommended Mitigation
Add control like ``onlyOwner()`` 

##

## [L-16] ``oracle.getAssetPrice(collateralAsset)`` this will revert some cases if ``collateralAsset`` is not exists 

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L367

##

## [L-17] There no ``deadline`` and no ``slippage protections`` when swapping tokens 

### Impact

The swapTokens function does not have a deadline or slippage protection. This means that there is no guarantee that the swap will be executed, and the user may lose money if the price of the tokens changes significantly during the swap.

A deadline is a time limit that is set for a transaction. If the transaction is not executed before the deadline, it is reverted. This can be useful for preventing users from losing money if the price of the tokens changes significantly during the swap.

Slippage protection is a feature that limits the amount of slippage that can occur during a swap. Slippage is the difference between the expected price of a swap and the actual price of the swap. If the price of the tokens changes significantly during the swap, the user may lose money due to slippage

### POC

https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L439-L456

### Recommended Mitigation

Add ``minOutAmount`` and ``deadline`` in ``swapTokens()`` functions 









