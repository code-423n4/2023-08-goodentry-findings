# Findings Summary

| ID     | Title                                                                   | Severity |
| ------ | ----------------------------------------------------------------------- | -------- |
| [L-01] | modifyTick did not check that the tick is properly ordered              | Low      |
| [L-02] | FixedOracle roundId should use the L2 block number                      | Low      |
| [L-03] | OracleConvert latestAnswer may overflow                                 | Low      |
| [L-04] | Should be compatible with the zero-value transfer revert                | Low      |
| [L-05] | claimFee should not use 1% of balance as a threshold                    | Low      |
| [L-06] | TokenisableRange deposit should check lpAmt is not zero                 | Low      |
| [L-07] | The FixedOracle update may conflict when the coin price is volatile     | Low      |
| [L-08] | Set deadline to block.timestamp has no effect                           | Low      |
| [L-09] | Hard-coded slippage cannot be modified in subsequent governance         | Low      |

# Detailed Findings

# [L-01] modifyTick did not check that the tick is properly ordered

## Description

```solidity
  function modifyTick(address tr, uint index) public onlyOwner {
    (ERC20 t0,) = TokenisableRange(tr).TOKEN0();
    (ERC20 t1,) = TokenisableRange(tr).TOKEN1();
    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
    ticks[index] = TokenisableRange(tr);
    emit ModifyTick(tr, index);
  }
```

modifyTick did not check that the tick is properly ordered. If the owner enters an incorrect tick, the ordering is damaged, and the correct tick cannot be found to complete the order.

## Recommendations

Check the tick is properly ordered when modifyTick

# [L-02] FixedOracle roundId should use the L2 block number

## Description

```solidity
  function latestRoundData()
      external
      view
      returns (
          uint80 roundId,
          int256 answer,
          uint256 startedAt,
          uint256 updatedAt,
          uint80 answeredInRound
      )
  {
      roundId = uint80(block.number);
      answer = hardcodedPrice;
      startedAt = block.timestamp;
      updatedAt = block.timestamp;
      answeredInRound = roundId;
  }
```
In arbitrum, `block.number` obtains the block number of L1, and the block interval is `12s`. 
If the owner executes `setHardcodedPrice`, there are two prices in the same roundId, and the price conflict lasts for `12s`.
By using L2 blocks, interval is only `1s`, this problem can be greatly alleviated.

## Recommendations

Use l2 block number

# [L-03] OracleConvert latestAnswer may overflow

## Description

```solidity
  function latestAnswer() public view returns (int256) {
    uint priceA = uint(getAnswer(CL_TOKENA));
    uint priceB = uint(getAnswer(CL_TOKENB));
    // @audit CL_TOKENB.decimals() + CL_TOKENA.decimals() < 8
    return int(priceA * priceB / (10 ** (CL_TOKENB.decimals() + CL_TOKENA.decimals() - 8))); 
  }
```
There is no rule that `CL_TOKENB.decimals() + CL_TOKENA.decimals()` must be greater than 8, so subtraction may overflow

## Recommendations

Not use subtraction, but `priceA * priceB * 10 ** 8 / (10 ** (CL_TOKENB.decimals() + CL_TOKENA.decimals()))`

# [L-04] Should be compatible with the zero-value transfer revert

## Description

```solidity
  // @audit Some token zero-value transfers will revert
  TOKEN0.token.transferFrom(msg.sender, address(this), n0);
  TOKEN1.token.transferFrom(msg.sender, address(this), n1);
  
  uint newFee0; 
  uint newFee1;
  // Calculate proportion of deposit that goes to pending fee pool, useful to deposit exact amount of liquidity and fully repay a position
  // Cannot repay only one side, if fees are both 0, or if one side is missing, skip adding fees here
    // if ( fee0+fee1 == 0 || (n0 == 0 && fee0 > 0) || (n1 == 0 && fee1 > 0) ) skip  
    // DeMorgan: !( (n0 == 0 && fee0 > 0) || (n1 == 0 && fee1 > 0) ) = !(n0 == 0 && fee0 > 0) && !(n0 == 0 && fee1 > 0)
  // @audit From here you can see that n0 == 0 is supported
  if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) ){
    address pool = V3_FACTORY.getPool(address(TOKEN0.token), address(TOKEN1.token), feeTier * 100);
    (uint160 sqrtPriceX96,,,,,,)  = IUniswapV3Pool(pool).slot0();
    (uint256 token0Amount, uint256 token1Amount) = LiquidityAmounts.getAmountsForLiquidity( sqrtPriceX96, TickMath.getSqrtRatioAtTick(lowerTick), TickMath.getSqrtRatioAtTick(upperTick), liquidity);
    if (token0Amount + fee0 > 0) newFee0 = n0 * fee0 / (token0Amount + fee0);
    if (token1Amount + fee1 > 0) newFee1 = n1 * fee1 / (token1Amount + fee1);
    fee0 += newFee0;
    fee1 += newFee1; 
    n0   -= newFee0;
    n1   -= newFee1;
  }
```
For some tokens, such as BNB, zero-value transfers will revert.

## Recommendations

transfer only when the quantity is not zero

# [L-05] claimFee should not use 1% of balance as a threshold

## Description

```solidity
  // If accumulated more than 1% worth of fees, compound by adding fees to Uniswap position
  if ((fee0 * 100 > bal0 ) && (fee1 * 100 > bal1)) { 
```

claimFee uses 1% of the balance as a threshold to consider whether to add fee to the position.   
However, the problem is that the balance may be very large. For a pool with deep liquidity, the fee cannot reach 1% for a long time.   
Because fee has been idle, resulting in low utilization rate and low income.

## Recommendations

Should use USD value to decide whether to add liquidity, not balance.

# [L-06] TokenisableRange deposit should check lpAmt is not zero

## Description

```solidity
  lpAmt = totalSupply() * newLiquidity / (liquidity + feeLiquidity); 
  liquidity = liquidity + newLiquidity;
  
  _mint(msg.sender, lpAmt);
```

When calculating the lp amount, if the number of tokens deposited by the user is low and the liquidity is large, the calculated lpAmt may be 0, and the user loses funds.
Combined with an attack similar to ERC4626, a malicious user may frontrun to deposit a large amount of liquidity, causing the lpAmt to calculate 0.
Of course, since totalSupply is initialized as 1e18, this kind of cost is high, the possibility is low, and the benefit is also low. But for the sake of the user, there should check lpAmt > 0.

## Recommendations

check lpAmt > 0

# [L-07] The FixedOracle update may conflict when the coin price is volatile

## Description

```solidity
  function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {
    hardcodedPrice = _hardcodedPrice;
    emit SetHardcodedPrice(_hardcodedPrice);
  }
```

When the coin price fluctuates dramatically, the fixedOracle update interval is very small.   
If there are two update txs at the same time, their execution will conflict, possibly resulting in a stale price

## Recommendations

Pass update time through the parameter, and only the latest timestamp price can be updated

# [L-08] Set deadline to block.timestamp has no effect

## Description

The protocol uses a lot of block.timestamp as deadline for external interactions such as swap, but this has no effect in practice because deadline is the time when tx is executed and is always valid.

## Recommendations

Pass deadline through the parameter.

# [L-09] Hard-coded slippage cannot be modified in subsequent governance

## Description

```solidity
uint[] memory amounts = ammRouter.swapExactTokensForTokens(
  amount, 
  getTargetAmountFromOracle(oracle, path[0], amount, path[1]) * 99 / 100, // allow 1% slippage 
  path, 
  address(this), 
  block.timestamp
);
```

The protocol uses a lot of hard-coded slippage, which cannot be modified in subsequent governance.     
In particular, the swap slippage is hard-coded to 1%, oracle prices are taken from DEX and CEX, and CEX is dominated. Chainlink oracle also has an update threshold of 0.5%.    
The uniswap pool token price is likely to be more than 1% different from the price of chainlink oracle, resulting in the swap being DOS for a period of time until the difference returns to 1%.    

## Recommendations

Use variable slippage