https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L182

The comment says ```Liquidate up to 50% of an unhealthy position``` but there is no Logic implemented in the function liquidate().

If this is just a commenting issue it should be corrected but if the protocol says the positions can only be liquidated upto 50% of the position maintained it should implement that in the code.