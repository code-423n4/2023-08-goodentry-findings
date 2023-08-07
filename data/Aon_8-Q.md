Block.timestamp should not be used as a deadline when making swaps with Uniswap.


## Impact: 

Transactions initiated by calling executeOperation(), close(), swapTokens() can be left lingering in the mempool for longer periods than users might desire.  


## The Vulnerability in Deatail:

The 3 external functions above ultimately call either swapExactTokensForTokens() or swapTokensForExactTokens().  Both of these initiate the swap (see the lines of code below).

Links to the two function with the vulnerability:
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L480
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L501

    function swapExactTokensForTokens(IUniswapV2Router01 ammRouter, IPriceOracle oracle, uint amount, 
    address[] memory path) internal returns (uint256 received)
    {
    if (amount > 0 && AmountsRouter(address(ammRouter)).getAmountsOut(amount, path)[1] > 0){
     checkSetAllowance(path[0], address(ammRouter), amount);
     uint[] memory amounts = ammRouter.swapExactTokensForTokens(
       amount,
       getTargetAmountFromOracle(oracle, path[0], amount, path[1]) * 99 / 100, // allow 1% slippage 
       path,
       address(this),
       Block.timestamp ✅
     );
     received = amounts[1];
     }
    }
  

Passing in block.timestamp as the deadline doesn’t actually initiate a deadline for the transaction.  This is because the check within Uniswap will alway pass when block.timestamp is passed in, regardless of how long the transaction sits in the mempool (the line of code below from Uniswap illustrates this concept).   


    require(deadline >= block.timestamp);
    // assume deadline is block.timestamp from the user input
    // it will always pass this check 


This is likely not a medium vulnerability under the current implementation because the slippage rate is hard-coded at 1%.  Due to this, users aren’t likely to lose more than 1% of their funds if the transaction takes a long time to process and the prices change.  However, some miners can theoretically hold the transaction maliciously  (the article below goes over this concept in more detail).  

https://blog.bytes032.xyz/p/why-you-should-stop-using-block-timestamp-as-deadline-in-swaps


## Solution:

The best solution is for users to input a deadline into swapExactTokensForTokens() or swapTokensForExactTokens().  The deadline would first need to be passed into executeOperation(),      close(), or swapTokens() and then passed into the two internal swap functions (see example below).

    function swapExactTokensForTokens(IUniswapV2Router01 ammRouter, IPriceOracle oracle, uint amount, 
    uint deadline ✅, address[] memory path) internal returns (uint256 received)
    {
    if (amount > 0 && AmountsRouter(address(ammRouter)).getAmountsOut(amount, path)[1] > 0){
     checkSetAllowance(path[0], address(ammRouter), amount);
     uint[] memory amounts = ammRouter.swapExactTokensForTokens(
       amount,
       getTargetAmountFromOracle(oracle, path[0], amount, path[1]) * 99 / 100, // allow 1% slippage 
       path,
       address(this),
       deadline ✅
     );
     received = amounts[1];
     }
    }



The front end functionality could be similar to what Uniswap implements.  In Uniswap, the user chooses the transaction deadline (5 min, 10min, etc..).  This deadline gets added to the block.timestamp of when the transaction is initiated, not mined.  The block.timestamp can be received on the front end with the help of ether.js.  Since the user input is added to the block.timestamp of when the transaction is initiated, a proper deadline can be set for when the transaction needs to be mined.  This scenario is illustrated below.

// basic formula for front end

    let userDeadline;
    // assume userDeadline is 15 (minutes).. inputted directly from user on the front end

    let deadline;
    // this variable is computed below, and is what gets passed as a parameter into the smart contract.

    deadline = block.timestamp + 60 * userDeadline;
    // block.timestamp is when the transaction gets initiated, and the userDeadline gets added to it after being translated into seconds
    // this value ultimately makes its way to the below check within Uniswap


// In Uniswap

    require(deadline >= block.timestamp);
    // The block.timestamp here is when the transaction gets mined
    // assume that deadline is set at 15 minutes from when transaction is initiated
    // If it takes 12 minutes for the transaction to get mined, then the deadline will be 180 seconds higher then the block.timestamp, and the check will pass
    // If it takes 16 minutes for the transaction to get mined, then the deadline will be 60 seconds lower then the block.timestamp, and the check will fail


