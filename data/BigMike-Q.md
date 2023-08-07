# LOW and NON-CRITICAL FINDINGS

### Instance Found
8... *7 low, 1 non-critical*

### Findings
from lines 16 to 21, in the OptionsPositionManager.sol file, it is recommended to use at least 3 indexed data in each event for on-chain retrieval. Using the indexed keyword for value types such as uint, bool, and address can also save gas costs.
[https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L16-L21]

in lines 392 and 440, *IPriceOracle oracle* is a reductant code, it is declared but unused.
[https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L392]

[https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L440]

In function *sellOptions* in line 384, the event comes before interaction. it is recommended to follow the Check, Effect, and Interaction method to avoid wrong output and reentrancy.
[https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L384-L405]

### Resolution
### L16-21
   event BuyOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);
  event SellOptions(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);
  event ClosePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);
  event LiquidatePosition(address indexed user, address indexed asset, uint amount, uint amount0, uint amount1);
  event ReducedPosition(address indexed user, address indexed asset, uint amount);
  event Swap(address indexed user, address sourceAsset, uint sourceAmount, address targetAsset, uint targetAmount);

### L392 and L440
    (ILendingPool LP, ,IUniswapV2Router01 router, address token0, address token1) = getPoolAddresses(poolId);
 
### L384 - L405
    PMWithdraw(LP, msg.sender, token0, amount0);
    PMWithdraw(LP, msg.sender, token1, amount1);
    checkSetAllowance(token0, optionAddress, amount0);
    checkSetAllowance(token1, optionAddress, amount1);
    uint deposited = TokenisableRange(optionAddress).deposit(amount0, amount1);
    
    cleanup(LP, msg.sender, optionAddress);
    cleanup(LP, msg.sender, token0);
    cleanup(LP, msg.sender, token1);
  
    emit SellOptions(msg.sender, optionAddress, deposited, amount0, amount1 );}