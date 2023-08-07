[G-1] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS
In Solidity, declaring a variable inside a loop can result in higher gas costs compared to declaring it outside the loop. This is because every time the loop runs, a new instance of the variable is created, which can lead to unnecessary memory allocation and increased gas costs

On the other hand, if you declare a variable outside the loop, the variable is only initialized once, and you can reuse the same instance of the variable inside the loop. This approach can save gas and optimize the execution of your code.

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L301
for (uint k = 0; k < ticks.length; k++){
      TokenisableRange t = ticks[k];
      address aTick = lendingPool.getReserveData(address(t)).aTokenAddress;

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L393C4-L395C36
 for(uint k=0; k<ticks.length; k++){
      TokenisableRange t = ticks[k];
      uint bal = getTickBalance(k);

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L71C4-L75C6
 for ( uint8 k = 0; k<assets.length; k++){
      address asset = assets[k];
      uint amount = amounts[k];
      withdrawOptionAssets(poolId, asset, amount, sourceSwap[k], user);
    }

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L93-L105C58
    for ( uint8 k =0; k<assets.length; k++){
      address debtAsset = assets[k];
      
      // simple liquidation: debt is transferred from user to liquidator and collateral deposited to roe
      uint amount = amounts[k];
      
      // liquidate and send assets here
      checkSetAllowance(debtAsset, address(lendingPool), amount);
      lendingPool.liquidationCall(collateral, debtAsset, user, amount, false);
      // repay tokens
      uint debt = closeDebt(poolId, address(this), debtAsset, amount, collateral);
      uint amt0 = ERC20(token0).balanceOf(address(this));
      uint amt1 = ERC20(token1).balanceOf(address(this));

```

[G-2] USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L92
    uint[2] memory amts = [ERC20(token0).balanceOf(address(this)), ERC20(token1).balanceOf(address(this))];

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L138
      address[] memory path = new address[](2);

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L171
    uint[] memory flashtype = new uint[](options.length);

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L202
    uint[] memory flashtype = new uint[](options.length);

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L294
      address[] memory path = new address[](2);

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L448
    address[] memory path = new address[](2);

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L493
    uint [] memory amountsIn = AmountsRouter(address(ammRouter)).getAmountsIn(recvAmount, path);

```

[G-3] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD
External functions do not require a context switch to the EVM, while public functions do.

Its possible to save 10-15 gas using external instead public for every function call

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L125C2-L127C95
 function name()     public view virtual override returns (string memory) { return _name; }
  /// @notice Get the symbol of this contract token
  function symbol()   public view virtual override returns (string memory) { return _symbol; }

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L101C2-L104C4
 function setEnabled(bool _isEnabled) public onlyOwner { 
    isEnabled = _isEnabled; 
    emit SetEnabled(_isEnabled);
  }

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L108-L111C4
  function setTreasury(address newTreasury) public onlyOwner { 
    treasury = newTreasury; 
    emit SetTreasury(newTreasury);
  }

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L116C2-L116C51
 function pushTick(address tr) public onlyOwner {

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L137-L137C52
  function shiftTick(address tr) public onlyOwner {

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L167-L167C65
  function modifyTick(address tr, uint index) public onlyOwner {
```

[G-4] SPLITTING REQUIRE() STATEMENTS THAT USE && SAVES GAS
Instead of using the && operator in a single require statement to check multiple conditions, using multiple require statements with 1 condition per require statement will save 30 GAS per &&

```
https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L209
      require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L271
      require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L108C3-L108C93
  require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L167
    require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L523
    require ( priceAssetA > 0 && priceAssetB > 0, "OPM: Invalid Oracle Price");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol#L536
    require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L170
    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

```

