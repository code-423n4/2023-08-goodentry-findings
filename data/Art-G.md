  ```
function executeOperation(
    address[] calldata assets,
    uint256[] calldata amounts,
    uint256[] calldata premiums,
    address initiator,
    bytes calldata params
  ) override external returns (bool result) {
    uint8 mode = abi.decode(params, (uint8) );
    // Buy options
    if ( mode == 0 ){
      (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));
      executeBuyOptions(poolId, assets, amounts, user, sourceSwap);
    }
    // Liquidate
    else {
      (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));
      executeLiquidation(poolId, assets, amounts, user, collateral);
    }
    result = true;
  }
```
In the above function in OptionsPositionManager.sol contract, there is no validation implemented to check the length of assets, amounts, and premiums arrays. This function calls executeBuyOptions and executeLiquidation internal functions with these parameters, but no validations in those internal functions as well. 

Note: premiums parameter is not used in the function.

It is recommended to have length validation as below to optimise the gas consumption and avoid to go further into the loop and assign value incorrectly to asset and amount.

```
require(assets.length == amounts.length == premiums.length, "OPM: Array Length Mismatch");
```
