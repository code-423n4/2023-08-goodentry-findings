During `withdraw` function (withdraw function has reentrancy guard), they can run `rebalance` and this can cause reentrancy attack, so I suggest adding nonReentrancy modifier to `rebalance` function

POC could not be written due to time and therefore classified as QA


```solidity
contracts/GeVault.sol:
  200:   /// @notice Rebalance tickers
  201:   /// @dev Provide the list of tickers from 
  202:   function rebalance() public {
  203:     require(poolMatchesOracle(), "GEV: Oracle Error");
  204:     removeFromAllTicks();
  205:     if (isEnabled) deployAssets();
  206:   }
 




  214:   function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {
  215:     require(poolMatchesOracle(), "GEV: Oracle Error");
  216:     if (liquidity == 0) liquidity = balanceOf(msg.sender);
  217:     require(liquidity <= balanceOf(msg.sender), "GEV: Insufficient Balance");
  218:     require(liquidity > 0, "GEV: Withdraw Zero");
  219:     
  220:     uint vaultValueX8 = getTVL();
  221:     uint valueX8 = vaultValueX8 * liquidity / totalSupply();
  222:     amount = valueX8 * 10**ERC20(token).decimals() / oracle.getAssetPrice(token);
  223:     uint fee = amount * getAdjustedBaseFee(token == address(token1)) / 1e4;
  224:     
  225:     _burn(msg.sender, liquidity);
  226:     removeFromAllTicks();
  227:     ERC20(token).safeTransfer(treasury, fee);
  228:     uint bal = amount - fee;
  229: 
  230:     if (token == address(WETH)){
  231:       WETH.withdraw(bal);
  232:       payable(msg.sender).transfer(bal);
  233:     }
  234:     else {
  235:       ERC20(token).safeTransfer(msg.sender, bal);
  236:     }
  237:     
  238:     // if pool enabled, deploy assets in ticks, otherwise just let assets sit here until totally withdrawn
  239:     if (isEnabled) deployAssets();
  240:     emit Withdraw(msg.sender, token, amount, liquidity);
  241:   }

```