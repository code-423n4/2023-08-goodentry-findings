## [Low] PositionManager.checkSetAllowance can overflow & make the functions using allowance unusable
The function `PositionManager.checkSetAllowance` increases the current allowance of a given address by `type(uint256).max`. When this allowance is non-zero at the time of the call, the increase will cause an overflow because the max uint256 is added to the pre-approved allowance. Consider using `safeApprove` instead, because this does not operate a sum that can overflow.

## [Low] RangeManager.removeFromStep reverts if one between the ticker and the range is not present in the lending pool
In RangeManager.removeFromStep the code assumes that both TokenisedRanges are initialized and added to the lending pool. This may not be true as for certain ranges, only the range or the ticker may be interesting, so it may be worth adding a check on whether `LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress == address(0)` and `LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress == address(0)`

## [Low] TokenisableRange.deposit should use `safeTransferFrom` instead of `transferFrom`
Using `safeTransferFrom` in [TokenisableRange's deposit function](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L222-L232) (similarly to how it's done in other parts of the codebase) ensures the call does not when dealing with non-standard tokens:
```Solidity
  function deposit(uint256 n0, uint256 n1) external nonReentrant returns (uint256 lpAmt) {
    // Once all assets were withdrawn after initialisation, this is considered closed
    // Prevents TR oracle values from being too manipulatable by emptying the range and redepositing 
    require(totalSupply() > 0, "TR Closed"); 
    
    claimFee();
-    TOKEN0.token.transferFrom(msg.sender, address(this), n0);
-    TOKEN1.token.transferFrom(msg.sender, address(this), n1);
+    TOKEN0.token.safeTransferFrom(msg.sender, address(this), n0);
+    TOKEN1.token.safeTransferFrom(msg.sender, address(this), n1);

    uint newFee0; 
    uint newFee1;
```

## [Low] RangeManager's constructor does not check for `asset0` and `asset1` 
Consider adding a check for these to not equal `address(0)`.

## [Informational] FixedOracle.sol contains the HardcodedPriceOracle contract. Consider matching these names
It's a best practice to match the [contract name](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/FixedOracle.sol#L3) with the Solidity file name. Consider renaming either the file or the contract.

## [Informational] FixedOracle.sol has no licensing information
Consider adding licensing info to [FixedOracle.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/FixedOracle.sol#L1).

## [Informational] Documentation of TokenisableRange.returnExpectedBalance is misleading
The `returnExpectedBalance` documentation in comment mentions the balance excludes fees, which are in fact included:
```diff
-  /// @notice Calculate the balance of underlying assets based on the assets price, excluding fees
+  /// @notice Calculate the balance of underlying assets based on the assets price, including fees
  function returnExpectedBalance(uint TOKEN0_PRICE, uint TOKEN1_PRICE) public view returns (uint256 amt0, uint256 amt1) {
    (amt0, amt1) = returnExpectedBalanceWithoutFees(TOKEN0_PRICE, TOKEN1_PRICE);
    amt0 += fee0;
    amt1 += fee1;
  }
```

## [Informational] V3Proxy does not need to inherit from the NonReentrant contract
V3Proxy does not seem to use the `nonReentrant` modifier, so it does not need to inherit from the `NonReentrant` contract

## [Informational] `for` loop in `OptionsPositionManager.liquidate` can be removed
When OptionsPositionManager prepares its call to the LP to liquidate a position, [it fills an array with zeros via a `for` loop](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L203). This is an unnecessary step because [the values are already zero at initialization](https://docs.soliditylang.org/en/v0.8.20/types.html#allocating-memory-arrays).

