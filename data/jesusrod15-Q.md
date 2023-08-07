title: many ticks can block the  function shiftTick(address tr) 
```c
 function shiftTick(address tr) public onlyOwner {
    TokenisableRange t = TokenisableRange(tr);
    (ERC20 t0,) = t.TOKEN0();
    (ERC20 t1,) = t.TOKEN1();
    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
    if (ticks.length == 0) ticks.push(t);
    else {
      // Check that tick is properly ordered
      if (!baseTokenIsToken0) 
        require( t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");
      else 
        require( t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");
      
      // extend array by pushing last elt
      ticks.push(ticks[ticks.length-1]);//@audit can reached index  limit array  gas ?
      // shift each element
      if (ticks.length > 2){ //@audit can reached index  limit array 
        for (uint k = 0; k < ticks.length - 2; k++) 
          ticks[ticks.length - 2 - k] = ticks[ticks.length - 3 - k];
        }
      // add new tick in first place
      ticks[0] = t;
    }
    emit ShiftTick(tr);
  }
```
link:https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L154

Due to the fact that the function shiftTick interacts in a for loop with the array called ticks, if the length of this array reaches the gas block limit, this function does not work anymore 
