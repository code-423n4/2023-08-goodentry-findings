# [L-01] `TokenisableRange.initProxy()` may be transacted by another person, like a frontrunner.

`TokenisableRange.initProxy()` sets the `creator = msg.sender` and changes the `status = ProxyState.INIT_LP`, the `creator` can not be changed after this transaction. But `TokenisableRange.initProxy()` can transact by anyone when the contract is deployed, so I think the better idea is to set the `creator = msg.sender` in the contract constructor and then check the creator in `TokenisableRange.initProxy()`.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L85-L89

# [L-02] Contracts never check the `RoePool.isDeprecated`.
[GeVault.constructor()](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L83) and [PositionManager.getPoolAddresses()](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L67) can use deprecated pools for further execution.



