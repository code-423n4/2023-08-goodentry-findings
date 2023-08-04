**AggregatorV3Interface.sol**
- L7/9/11 - Multiple functions are created within the interface, which are never used within the project, they should be removed.


**TokenisableRange.sol**
- L13/14 - IAaveOracle is imported twice, one of the two imports should be eliminated.

- L54/55 - Variables are created in storage that are never used, therefore they should be eliminated.


**FixedOracle.sol**
- L11 - There are requirements with messages of more than 32 bytes, this generates extra gas that could be reduced with shorter messages.


**DataTypes.sol**
- L48 - The InterestRateMode enum is created, but it is never used throughout the contract, therefore it should be eliminated, since it generates an unnecessary gas expense.


**OptionsPositionManager.sol**
- L316/320 - - When carrying out a previous validation, the unchecked operation could be carried out, since there would be no way to generate underflow.

- L533/544 - There are multiple functions that could be viewed, since they do not modify the state of the contract.
