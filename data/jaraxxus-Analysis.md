### Codebase quality analysis
- Codebase is well written, just missing a couple low-hanging fruits like returning excess msg.value and checking stale prices in Chainlink's aggregator feeds (check for round completeness, updatedPrice, heartbeat price)

```
GeVault.sol
    if (msg.value > 0){
      require(token == address(WETH), "GEV: Invalid Weth");
      // wraps ETH by sending to the wrapper that sends back WETH
      WETH.deposit{value: msg.value}();
      amount = msg.value;
    }
    else { 
      ERC20(token).safeTransferFrom(msg.sender, address(this), amount);
```
```
LPOracle.sol
  function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {
    (
      , 
      int price,
      ,
      uint timeStamp,
    ) = priceFeed.latestRoundData();
    require(timeStamp > 0, "Round not complete");
    return price;
  }
```
 
### Centralization risks
- Owner has some privilege in the GeVault, such as setting the TvlCap, base fee, and pool. 
- Base fee can be set to a maximum of 100% in basis points, which is an extremely high value

```
GeVault.sol
  /// @notice Set the base fee
  /// @param newBaseFeeX4 New base fee in E4
  function setBaseFee(uint newBaseFeeX4) public onlyOwner {
  require(newBaseFeeX4 < 1e4, "GEV: Invalid Base Fee");
    baseFeeX4 = newBaseFeeX4;
    emit SetFee(newBaseFeeX4);
  }
```

- Owner controls which pools to use and can deprecate any pool at any time

```
RoeRouter.sol
  function deprecatePool(uint poolId) public onlyOwner {
    pools[poolId].isDeprecated = true;
    emit DeprecatePool(poolId);
  }
```

### Systemic risks
- Usage of external oracles like Chainlink and without a backup oracle may be risky if chainlink fails
- Deploying on L2 means having to check the sequencer uptime 
- Usage of slot0 can be dangerous, recommend using TWAPs (observe method) to get the sqrtPriceX96 instead.
- Heavy reliance on oracle decimals, which Chainlink may change any time

### Mechanism review
- Uniswap Minting, increasing liquidity and decreasing liquidity is incorporated well
- Use of Chainlink oracle is decent, although need to check the output of the Oracle

### Time spent:
10 hours