### [G-01] With assembly, .call (bool success) transfer can be done gas-optimized

When using assembly language, it is possible to call the transfer function of an Ethereum contract in a gas-optimized way by using the .call function with specific input parameters. The .call function takes a number of input parameters, including the address of the contract to call, the amount of Ether to transfer, and a specification of the gas limit for the call. By specifying a lower gas limit than the default, it is possible to reduce the gas cost of the transfer.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L156C7-L156C60

```solidity
File: /contracts/helper/V3Proxy.sol
156: msg.sender.call{value: msg.value - amounts[0]}("");
174: payable(msg.sender).call{value: amountOut}("");
192: payable(msg.sender).call{value: amounts[1]}("");
```

### [G-02] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L72C6-L73C32

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol
72 : address asset = assets[k];
73:      uint amount = amounts[k];
94: address debtAsset = assets[k];
97: uint amount = amounts[k];
```

### [G-03] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

per 50

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol
138: address[] memory path = new address[](2);
171: uint[] memory flashtype = new uint[](options.length);
202: uint[] memory flashtype = new uint[](options.length);
448: address[] memory path = new address[](2);
```

### [G-04] Using calldata instead of memory for read-only arguments in external functions saves gas

When a function is called externally in Solidity, the arguments are passed in the calldata, which is a special area of memory that is used to store the function arguments and any other data that is required for the function call. By using calldata instead of memory for read-only arguments, we can avoid the need for memory allocation and copy operations, which can reduce the gas cost of the contract.

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol
159: function buyOptions(
160:    uint poolId, 
161:    address[] memory options, 
162:    uint[] memory amounts, 
163:    address[] memory sourceSwap
164:  )

189: function liquidate (
190:    uint poolId, 
191:    address user,
192:    address[] memory options, 
193:    uint[] memory amounts,
194:    address collateralAsset
195  )
```

### [G-05] Use emit outside the loop for better gas optimization**.**

Emitting an event inside a loop performs a LOG op N times, where N is the loop length. Consider refactoring the code to emit the event only once at the end of loop. Gas savings should be multiplied by the average loop length.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L106C7-L106C85

```solidity
File: /contracts/PositionManager/OptionsPositionManager.sol
106: emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1]);
```

### [G-06] **The result of function calls should be cached rather than re-calling the function**

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L269

```solidity
File: contracts/GeVault.sol
//@audit getTVL()
269: require(tvlCap > valueX8 + getTVL(), "GEV: Max Cap Reached");
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L292C3-L327C4

```solidity
File: contracts/TokenisableRange.sol
//@audit totalSupply()
292: function withdraw(uint256 lp, uint256 amount0Min, uint256 amount1Min) external nonReentrant returns (uint256 removed0, uint256 removed1) {
    claimFee();
    uint removedLiquidity = uint(liquidity) * lp / totalSupply();
    (removed0, removed1) = POS_MGR.decreaseLiquidity(
      INonfungiblePositionManager.DecreaseLiquidityParams({
        tokenId: tokenId,
        liquidity: uint128(removedLiquidity),
        amount0Min: amount0Min,
        amount1Min: amount1Min,
        deadline: block.timestamp
      })
    );
    liquidity = uint128(uint256(liquidity) - removedLiquidity); 
    
    POS_MGR.collect( 
      INonfungiblePositionManager.CollectParams({
        tokenId: tokenId,
        recipient: msg.sender,
        amount0Max: uint128(removed0),
        amount1Max: uint128(removed1)
      })
    );
    // Handle uncompounded fees
    if (fee0 > 0) {
      TOKEN0.token.safeTransfer( msg.sender, fee0 * lp / totalSupply());
      removed0 += fee0 * lp / totalSupply();
      fee0 -= fee0 * lp / totalSupply();
    } 
    if (fee1 > 0) {
      TOKEN1.token.safeTransfer(  msg.sender, fee1 * lp / totalSupply());
      removed1 += fee1 * lp / totalSupply();
      fee1 -= fee1 * lp / totalSupply();
    }
    _burn(msg.sender, lp);
    emit Withdraw(msg.sender, lp);
  }
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L352C2-L357C5

```solidity
File: contracts/TokenisableRange.sol
//@audit totalSupply()
352: function getValuePerLPAtPrice(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 priceX1e8) {
353:    if ( totalSupply() == 0 ) return 0;
354:    (uint256 amt0, uint256 amt1) = returnExpectedBalance(TOKEN0_PRICE, TOKEN1_PRICE);
355:    uint totalValue = TOKEN0_PRICE * amt0 / (10 ** TOKEN0.decimals) + amt1 * TOKEN1_PRICE / (10 ** TOKEN1.decimals);    return totalValue * 1e18 / totalSupply();
356:  }
```

- https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L377C3-L381C4

```solidity
File: contracts/TokenisableRange.sol
//@audit totalSupply()
377: function getTokenAmounts(uint amount) external view returns (uint token0Amount, uint token1Amount){
378:    (token0Amount, token1Amount) = getTokenAmountsExcludingFees(amount);
379:    token0Amount += fee0 * amount / totalSupply();
380:    token1Amount += fee1 * amount / totalSupply();
381:  }
```