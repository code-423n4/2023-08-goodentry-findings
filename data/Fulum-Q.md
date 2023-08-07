## Make the Uniswap V3 swaps functions more sure.
	
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L137-L145
	
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L147-L158

You can add this verification `require(msg.value > 0, "Must pass non 0 ETH amount");` for the `swapExactETHForTokens` and `swapETHForExactTokens` functions for avoid user make a call with 0 value.
	
## Add verification for deprecated pool in the `OptionsPositionManager.sol` contract 
	
In `OptionsPositionManager.sol` contract, a lot of functions use the function `getPoolAddresses(poolId)` which call `RoeRouter.sol` to get the pool address, the Uniswap router address, the tokens.
	
You have the `deprecatePool()` function in `RoeRouter.sol` to update the `isDeprecated` bool of the pool to true to significate that the pool is deprecated.
But the contract doesn't have a function to return the complete status of the pool (the `RoePool` struct in the contract). With a function like this you can add verification in the functions of the `OptionsPositionManager.sol` contract to verify if a pool is deprecated and avoid potentials bad using with a deprecated pool.
I didn't have enough time to see if this lack of verification could cause an issue, but you can avoid a factor risk by checking the status of the pool.