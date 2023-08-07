Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ILendingPoolConfigurator.sol




The ReserveInitialized event is emitted when a new reserve is initialized. The event has a parameter called asset that specifies the address of the underlying asset for the reserve. However, the current code has a typo in the asset parameter. The parameter is currently named asset_, but it should be named asset.



The steps to reproduce the bug are as follows:

Open the ILendingPoolConfigurator interface in a text editor.
Find the ReserveInitialized event.
Change the asset_ parameter to asset.

If we  do not make this change, the ReserveInitialized event will emit an event with the typoed asset_ parameter.

The suggested fix for the bug is to change the asset_ parameter to asset in the ReserveInitialized event. The documentation for the ILendingPoolConfigurator interface should also be updated to reflect the change.