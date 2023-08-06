1) PositionManager has two state variables that are never initialized or used in the contract.
   ```
    ILendingPool public LENDING_POOL; 
    RoeRouter public ROEROUTER; 
   ``` 

2) Aave lendingPool::repay function returns the final amount that was repaid. The remaining balance should be computed based on return value from the pool and now internal to the contract is a better approach,

```
 if ( debt > 0 ){
        if (amt <= debt ) {
          LP.repay( asset, amt, 2, user);
          return;
        }
        else {
          LP.repay( asset, debt, 2, user);
          amt = amt - debt;
        }
      }
      // deposit remaining tokens
      LP.deposit( asset, amt, user, 0 );
```

3) RangeManager contract is using older complier and also has floating pragma.

4) V2Proxy::swapExactTokensForTokens() has to parameter, but never used in the function.
   The same issue lies with other swap functions where to was not used.

5) FixedOracle contract is using floating pragma and different version of compiler.