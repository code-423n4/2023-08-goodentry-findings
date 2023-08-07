# 1. Codebase

## 1.1 overview
Good Entry is a derivatives market that provides liquidity.
It is built on top of ```Uniswap V3``` and ```Aave's lending market```. Moreover, it uses ```Chainlink Oracles```.

## 1.2 RoeRouter
RoeRouter contract holds ROE lending pools and ROE treasury.
It is an Ownable contract, and so only the owner can perform its operations.

RoeRouter is based on ```RoePool``` struct, which represents the lending pool.
The list of pools is saved into ```pools``` variable
The Owner can add a new pool with the given LP address provider and the address of AMM.

Moreover, the Owner can update RoeRouter.treasury address and set a pool as deprecated.

## 1.3 TokenisableRange
TokenisableRange is used to tokenize an [Uniswap V3 position](https://docs.uniswap.org/contracts/v3/reference/core/libraries/Position). To do so, it holds information about ```lowerTick``` and ```upperTick```.

Moreover, it holds ```liquidity``` which ```creator``` owns between lower and upper tick and methods to deposit and withdraw assets in this range.
Lastly, TokenisableRange has methods to claim fees and send them to treasury (that has a hardcoded address)

## 1.4 RangeManager
RangeManager provides several help functions to create and manage ```V3 TokenisableRange```.
It permits generating a new tokenisedRange (and relative tokenisedTicker) checking that it doesn't overlap 
with others.
Moreover, it permits removing assets from range+ticker and transferring assets from range to other.

## 1.5 GeVault
GeVault is an ```ERC20``` contract that holds TokenisableRange tickers.
It holds tickers inside ```ticks``` list.
It offers the function to add a new ticker (```pushTick```), shift all tickers and put a new ticker at position 0 (```shiftTick```), modify a ticker in a position with another (```modifyTick```), rebalance ticker with respect
assets (```rebalance```), withdraw assets from a ticker (withdraw), and deposit assets into GeVault.

## 1.6 PositionManager
PositionManager implements ```IFlashLoanReceiver``` interface.
It contains base functions to implement ```OptionsPositionManager```

## 1.7 OptionsPositionManager
It contains flashloan logic.
It implements PositionManager, so it implements Aave compatible ```IFlashLoanReceiver``` interface.
Operations that can be executed with flashloan amount are inside ```executeOperation```.
A user can call two types of flashloan operation:

* ```executeBuyOptions```
* ```executeLiquidation```

Moreover, users can sell and withdraw a single option.

### 1.7.1 executeBuyOptions
Using ```executeBuyOptions()```, a user can buy each option of the given assets list, calling ```withdrawOptionAssets``` for each asset.
withdrawOptionAssets() withdraw option asset and remove liquidity from the pool.

### 1.7.2 executeLiquidation
Using ```executeLiquidation()```, a user can liquidate each option of the given assets list, giving back collateral to Roe.


# 2. Centralization risks
As we reported in medium findings, GoodEntry seems to have some centralization risks.
Moreover, ```Owner``` sometimes seems to have too much power, and this could lead customers to untrust.

Inside ```RoeRouter```, the owner can change treasury addresses without any governance policy.
In ```GeVault```, the owner can change the pool status and modify the behavior of the entire vault.
Moreover, he/she can manage tickers how he wants, and with ```modifyTick``` he can modify any tick without any kind of bound.
Moreover, the owner can modify ```tvlCap``` and ```baseFee```.

We suggest developing a governance policy to give customers more peace of mind.

# 3. Inconsistent developing
There are parts in the codebase that seems old or not yet developed.
This could lead to an incoherence inside the codebase.

For example, in RoeRouter, there is ```getPoolsLength()``` which is never used.
Moreover, the owner can set a pool as ```deprecated```, but it seems to not affect contract behavior.
getPoolsLength(), for example, returns "all pools" length, and there is no way to know how many pools are not deprecated or to block operations on deprecated pools.
Furthermore, when a pool is deprecated, there is no way to go back. So, it seems it's not used, or it is not used yet.

```GeVault```, ```RoeRouter``` and ```TokenisableRange``` have treasury parameter. We think that it's the same parameter. If so, it could be better to hold it in only one contract, and if needed, update it in that contract.

Some important parameters, like ```isEnabled``` inside GeVault, should have more details on their use.
They are very delicate and must be updated carefully.

GeVault contract creates an instance of RangeManager but never uses it. It could lead to inconsistent behaviors, especially when developers update contracts.

However, tests were written very well, and this protects you from many errors.

# 4. Time Spent
* Day 0: Study of derivatives, Aave and Uniswap V3
* Day 1: Visual inspection and analysis of RoeRouter.sol, TokenisableRange.sol, and GeVault.sol. 
* Day 2: Visual inspection and analysis of PositionManager and OptionsPositionManager
* Day 3: Report writing

Time spent: 30 hours


### Time spent:
29 hours