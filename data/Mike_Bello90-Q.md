Contest Good Entry Protocol – QA Report.

No. 1: Oracle variable is not used in OptionsPositionManager – sellOptions.

The oracle variable in the sellOptions function is not used, so you can avoid to declaring this variable and save some memory.

https://github.com/GoodEntry-io/ge/blob/8a2686b14114edbd1ec523d79304ed678cc2e915/contracts/PositionManager/OptionsPositionManager.sol#L392C36-L392C42


No. 2:  Functions “sanityCheckUnderlying” & “addDust” can be declared as view functions.

https://github.com/GoodEntry-io/ge/blob/8a2686b14114edbd1ec523d79304ed678cc2e915/contracts/PositionManager/OptionsPositionManager.sol#L533

https://github.com/GoodEntry-io/ge/blob/8a2686b14114edbd1ec523d79304ed678cc2e915/contracts/PositionManager/OptionsPositionManager.sol#L544

No. 3: Contract GEVault – Function withdraw - Use call instead of Transfer on payable addresses.

Using transfer can cause the function to revert if the msg.sender is a contract that does not accept transfer, it is recommended to use call and reentrancy guard instead.

https://github.com/GoodEntry-io/ge/blob/8a2686b14114edbd1ec523d79304ed678cc2e915/contracts/GeVault.sol#L232


No. 4: Contract GEVault – Function checkSetApprove.- Don’t use Max allowance.

Consider changing the allowance given for this function from type(uint256).max to “amount”, so in case of a hack of one of the contracts that get the maximum allowance, it only can move the set “amount” of tokens.

https://github.com/GoodEntry-io/ge/blob/8a2686b14114edbd1ec523d79304ed678cc2e915/contracts/GeVault.sol#L386

The same recommendation is given for checkSetAllowance in PositionManager contract.

https://github.com/GoodEntry-io/ge/blob/8a2686b14114edbd1ec523d79304ed678cc2e915/contracts/PositionManager/PositionManager.sol#L81C38-L81C38
