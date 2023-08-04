**OptionsPositionManager.sol**
- L367/524 - A division is made by: oracle.getAssetPrice(collateralAsset), before executing the operation it should be validated that the oracle value is != 0, to handle the exception if it happens.

- L516/517/518/519/520 - There is commented code, which should be removed, since it is not used and generates a reduction in the understanding of the code.


**RangeManager.g**
- L190 - The name of the function in cleanup, but having two words would not be respecting snake case, the correct name should be: cleanUp().


**PoolAddress.sol**
- L38/39 - abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()
Use abi.encode() instead which will pad items to 32 bytes, which will prevent hash collisions (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.
If all arguments are strings and or bytes, bytes.concat() should be used instead.

- The contract is a library, but it is in the interfaces folder, this is confusing and should be modified, as it can cause confusion for programmers looking at the source code.


**PositionManager.sol**
- L23 - The variable ROEROUTER is created in storage, for a matter of convention, the name should preferably be separated with underscore: ROE_ROUTER.

- L89 - The name of the function in cleanup, but having two words would not be respecting snake case, the correct name should be: cleanUp().

- L143/144 - When carrying out a previous validation, the unchecked operation could be carried out, since there would be no way to generate underflow.


**V3Proxy.sol**
- L76/77/78 - No validation is performed in the constructor and the variables (ROUTER, QUOTER and feeTier) are immutable, therefore they should be validated before setting the variable to != 0x.


**GeVault.sol**
- L8 - IAaveLendingPoolV2 is imported, but never used.

- L62 - The constant nearbyRanges is created, as a matter of convention, the name should preferably be in uppercase: NEARBY_RANGES.

- L222/376 - A division is made by: oracle.getAssetPrice(token), before executing the operation it should be validated that the value of the oracle is != 0, to handle the exception if it happens.

- L247/255/261 - The deposit() function has two deposit mechanisms, one for gas and the other for tokens, the problem with this is that if one sets the two variables, send gas and put a different value to amount, to the user
It is not clear what is being executed, also making the amount input useless. The recommendation is to create two different functions, one for tokens and one for gas.
 
- L247 - The deposit() function has multiple logics inside, therefore they should use auxiliary functions to provide greater simplicity and readability in the code.
