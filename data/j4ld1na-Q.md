|     | Issues                                                                            | Instances |
| :-: | :-------------------------------------------------------------------------------- | --------: |
| 01  | The function `executeOperation` and `withdrawOptionAssets` always returns `true`. |         2 |
| 02  | Dont check value return.                                                          |         1 |
| 03  | `if`-statement can be converted to a ternary operator.                            |         2 |
| 04  | More than one space around an assignment or other operator to align with another. |        10 |

## 01 The function `executeOperation` and `withdrawOptionAssets` always returns `true`.

There are 2 instances for this issue:

_The functions `executeOperation` and `withdrawOptionAssets` always returns true. The last line result = true assigns the value true to the variable result, indicating that the operation execution is always considered successful, and therefore, the function always returns true._

_Regardless of the result of the internal operations, the function assigns true to the variable result, indicating that the execution was successful._

-   [OptionsPositionManager.sol#L38-L57](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L38-L57)

```
 function executeOperation(
    address[] calldata assets,
    uint256[] calldata amounts,
    uint256[] calldata premiums,
    address initiator,
    bytes calldata params
  ) override external returns (bool result) {
    uint8 mode = abi.decode(params, (uint8) );
    // Buy options
    if ( mode == 0 ){
      (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));
      executeBuyOptions(poolId, assets, amounts, user, sourceSwap);
    }
    // Liquidate
    else {
      (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));
      executeLiquidation(poolId, assets, amounts, user, collateral);
    }
    result = true;
  }
```

-   [OptionsPositionManager.sol#L123-L150](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L123-L150)

```
function withdrawOptionAssets(
        uint poolId,
        address flashAsset,
        uint256 flashAmount,
        address sourceSwap,
        address user
    ) private returns (bool result) {
        (
            ,
            IPriceOracle oracle,
            IUniswapV2Router01 router,
            address token0,
            address token1
        ) = getPoolAddresses(poolId);
        sanityCheckUnderlying(flashAsset, token0, token1);
        // Remove Liquidity and get underlying tokens
        (uint256 amount0, uint256 amount1) = TokenisableRange(flashAsset)
            .withdraw(flashAmount, 0, 0);
        if (sourceSwap != address(0)) {
            require(
                sourceSwap == token0 || sourceSwap == token1,
                "OPM: Invalid Swap Token"
            );
            address[] memory path = new address[](2);
            path[0] = sourceSwap;
            path[1] = sourceSwap == token0 ? token1 : token0;
            uint amount = sourceSwap == token0 ? amount0 : amount1;

            uint received = swapExactTokensForTokens(
                router,
                oracle,
                amount,
                path
            );
            // if swap underlying, then sourceSwap amount is 0 and the other amount is amount withdrawn + amount received from swap
            amount0 = sourceSwap == token0 ? 0 : amount0 + received;
            amount1 = sourceSwap == token1 ? 0 : amount1 + received;
        }
        emit BuyOptions(user, flashAsset, flashAmount, amount0, amount1);
        result = true;
    }
```

Recommended Mitigations Steps:

-   The current implementation of the function may not be appropriate if it is expected that the function could return false in case any of the internal operations fail or if any other error occurs. If it is necessary to distinguish between successful and failed executions and if it is desired to return false in specific cases, the implementation should be modified to reflect the desired and appropriate behavior according to the protocol rules or the specific logic of the contract.

## 02 Dont check value return.

_The `withdrawOptionAssets`function return a `Boolean` indicating whether the call succeeded or failed._

There an instance:

-   [OptionsPositionManager.sol#L123-L130](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L123-L130)

```
withdrawOptionAssets(poolId, asset, amount, sourceSwap[k], user);
```

## 03 `if`-statement can be converted to a ternary operator.

_The code can be made more compact while also increasing readability by converting the following if-statements to ternaries (e.g. `foo += (x > y) ? a : b`)_

1. Increases readability
2. Shortens the overall SLOC

There are 2 instances:

-   [OptionsPositionManager.sol#L549-L550](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L549-L550)

```
if (scale0 > scale1) amount = scale0;
else amount = scale1;
```

change to:

```
amount = scale0 > scale1 ? scale0 : scale1;
```

-   [GeVault.sol#L450-L453](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L450-L453)

```
if (increaseToken0)
      adjustedBaseFeeX4 = baseFeeX4 * value0 / (value1 + 1);
    else
      adjustedBaseFeeX4 = baseFeeX4 * value1 / (value0 + 1);
```

change to:

```
adjustedBaseFeeX4 = increaseToken0 ? baseFeeX4 * value0 / (value1 + 1)
                  : baseFeeX4 * value1 / (value0 + 1);
```

## 04 More than one space around an assignment or other operator to align with another.

As indicated in the [Docs-Solidity-Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#whitespace-in-expressions) example:

```
Yes:

x = 1;
y = 2;
longVariable = 3;



No:

x            = 1;
y            = 2;
longVariable = 3;
```

There are 10 instances:

-   [TokenisableRange.sol#L92-L95](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L92-L95)

```
    TOKEN0.token    = asset0;
    TOKEN0.decimals = asset0.decimals();
    TOKEN1.token     = asset1;
    TOKEN1.decimals  = asset1.decimals();
```

-   [TokenisableRange.sol#L103](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L103)

```
feeTier   = 5;
```

[TokenisableRange.sol#L108-L109](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L108-L109)

```
      _name     = string(abi.encodePacked("Ticker ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));
     _symbol    = string(abi.encodePacked("T-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));
```

[TokenisableRange.sol#L111-L115](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L111-L115)

```
      feeTier   = 5;
      _lowerTick = (_lowerTick + int24(feeTier)) - (_lowerTick + int24(feeTier)) % int24(feeTier * 2);
      _upperTick = (_upperTick + int24(feeTier)) - (_upperTick + int24(feeTier)) % int24(feeTier * 2);
      _name     = string(abi.encodePacked("Ranger ", baseSymbol, " ", quoteSymbol, " ", startName, "-", endName));
      _symbol   = string(abi.encodePacked("R-",startName,"_",endName,"-",baseSymbol,"-",quoteSymbol));
```

[TokenisableRange.sol#L243-L246](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L243-L246)

```
      fee0 += newFee0;
      fee1 += newFee1;
      n0   -= newFee0;
      n1   -= newFee1;
```
