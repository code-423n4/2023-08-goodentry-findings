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