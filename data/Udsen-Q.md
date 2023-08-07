## 1. ARRAY LENGTHS OF THE TWO ARRAYS SHOULD BE CHECKED FOR EQUALITY

The `OptionsPositionManager.executeBuyOptions` is used to buy the flashloaned options. The function is passed in two arrays representing flash borrowed asset address and flash borrowed asset amount as shown below:

```solidity
  function executeBuyOptions(
    uint poolId,
    address[] calldata assets,
    uint256[] calldata amounts,
```

The `assets.length` is used in a `for` loop to iterate over the elements of the both the arrays.
But their array lengths are not checked for equality before being used inside the functions. If the array lengths are different then the transaction will revert with array out of bound exception. This will waste lot of gas of the transaction. 

Hence it is recommended to perform the following check at the beginning of the `executeBuyOptions` function to verify that the the two arrays are of the same length. If not can revert the transaction at the beginning of the function, hence saving gas. 

    require( assets.length == amounts.length, "Array length should match");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L71-L75
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L93-L97

## 2. UNUSED INPUT VARIABLE NAMES IN FUNCTIONS CAN BE REMOVED

The `OptionsPositionManager.executeOperation` function is used to open a leverage positon or to liquidate a position. The following two parameters provided to the function as input variables are never used within the scope of the function execution.

    uint256[] calldata premiums,
    address initiator,

Hence the two variable names can be removed and only the data types can be left in place as shown below:

    uint256[] calldata ,
    address ,

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L41-L42

## 3. LACK OF INPUT VALIDATIONS IN THE CONSTRUCTOR OF CRITICAL CONTRACT

The `V3Proxy` contract initializes the state variables inside the contract constructor as shown below:

    constructor(ISwapRouter _router, IQuoter _quoter, uint24 _fee) {
        ROUTER = _router;
        QUOTER = _quoter;
        feeTier = _fee;
    
    }

But the input parameters to the `constructor` lack input data validation. This could lead to erroneous values assigned to the state of the contract during deployment. Hence it is recommended to add the neccesary input validations in the `V3Proxy.constructor` function.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L75-L80

## 4. ARITHMETIC OPERATION CAN BE UNCHECKED DUE TO PREVIOUS CONDITIONAL CHECK

In the `PositionManager.cleanup` function, any `debt` amount will be repaid back to `ROE`. In the process the `amt = amt - debt` operation is executed only if `amt > debt`. Hence the arithmetic operation can be `unchecked` to save gas.

          unchecked {amt = amt - debt;}

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol#L103  

## 5. RECOMMENDED TO EMIT BOTH OLD AND NEW VALUES FOR CRITICIAL STATE CHANGES

The `RoeRouter.setTreasury` function is used to modify the `treasury` to a new address. But the emitted log only contains the `newTreasury` address as shown below:

    emit UpdateTreasury(newTreasury);

It is recommended to include both new and old treasury addresses when emitting critical state changes.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L86
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L186

## 6. TYPO'S IN THE NATSPEC COMMENTS SHOULD BE CORRECTED

     /// @param token0 address of the `one` token of the pair 

Above comment should be corrected as follows:

     /// @param token0 address of the `first` token of the pair   

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L56

     /// @param amount Amount of `target token received`

Above comment should be corrected as follows:

     /// @param amount Amount of `source token to swap`  

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L467
