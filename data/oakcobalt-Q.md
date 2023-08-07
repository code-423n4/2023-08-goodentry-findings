### 1. Low -  Wrong token name encoded in public variable _name, and wrong token symbol encoded into public variable _symbol

In TokenisableRange.sol - `initProxy()`, wrong token names and symbols are assigned. 
```solidity
//TokenisableRange.sol
 function initProxy(IAaveOracle _oracle, ERC20 asset0, ERC20 asset1, uint128 startX10, uint128 endX10, string memory startName, string memory endName, bool isTicker) external {
...
|>      _name     = string(abi.encodePacked("Ticker ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));
|>      _symbol    = string(abi.encodePacked("T-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));
```
As seen from above, token symbol is wrongly assigned to `_name`, while token name is wrongly assigned to `_symbol`.

Recommendation:
Assign token name to `_name`, and assign token symbol to `_symbol`.

### 2. Low - No check on range validity in modifyTick 

In GeVault.sol - `modifyTick()`, there is no check enforced to ensure the modified tick range in the correct order. However, tick range order check is enforced in similar functions that modifies `ticks[]` such as `pushTick()` and `shiftTick()`.
```solidity
    function modifyTick(address tr, uint index) public onlyOwner {
        (ERC20 t0, ) = TokenisableRange(tr).TOKEN0();
        (ERC20 t1, ) = TokenisableRange(tr).TOKEN1();
        require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
|>      ticks[index] = TokenisableRange(tr);
        emit ModifyTick(tr, index);
    }
```
See how check of tick range order is enforced in `pushTick()`.
```solidity
    function pushTick(address tr) public onlyOwner {
...
if (baseTokenIsToken0)

                require(
                    t.lowerTick() > ticks[ticks.length - 1].upperTick(),
                    "GEV: Push Tick Overlap"
                );

            else
                require(
                    t.upperTick() < ticks[ticks.length - 1].lowerTick(),
                    "GEV: Push Tick Overlap"
                );

            ticks.push(TokenisableRange(tr));
...
```
Recommendation:
Enforce the same check in `modifyTick()` also.

### 3. Low - In TokenisableRange.sol iniProxy function, when isTicker is true, upperTick and LowerTick might be miscalculated, resulting in a range below input lowerTick to be initialized.

In TokenisableRange.sol -`initProxy()`, when `isTicker` is true, `upperTick` and `lowerTick` can be miscalculated due to incorrect rounding. 

Suppose this example, when the input `upperTick` (UT) is 17, and the input `lowerTick` (LT) is 11, then middle tick (MT) would be 14 based on current implementation. Then `upperTick` (UT) would be rounded as follows `UT= (14+5)- (14+5)%(2*5)=10` this results in a tick range `[0,10]` to be initialized. Note that 10 is actually smaller than `lowerTick` (11). 

Recommendation:
In `initProxy()` when `isTicker` is true, calculate `_upperTick` as follows
```solidity
int24 midleTick;
            midleTick = (_upperTick + _lowerTick) / 2;
            _upperTick =
                (midleTick + int24(feeTier)) -
                ((midleTick + int24(feeTier)) % int24(feeTier));
```
