## [L-01] Unused `to` parameter in `V3Proxy` swap functions
The swap functions in `V3Proxy` take an `address to` parameter intended to allow the caller to specify the swap destination; however, this parameter is never used. Instead, tokens are sent to the `msg.sender` which means that whilst the funds do not reach their intended destination, they are not lost (unless called by another contract which does not expect to receive funds and thus does not forward them on to the user).

## [L-02] Pool can be added multiple times in `RoeRouter::addPool`
In the following snippet, it is possible for the owner to add a pool multiple times:
```solidity
function addPool(
    address lendingPoolAddressProvider, 
    address token0, 
    address token1, 
    address ammRouter
  ) 
    public onlyOwner 
    returns (uint poolId)
  {
    require (
      lendingPoolAddressProvider != address(0x0) 
      && token0 != address(0x0) 
      && token1 != address(0x0) 
      && ammRouter != address(0x0), 
      "Invalid Address"
    );
    require(token0 < token1, "Invalid Order");
    pools.push(RoePool(lendingPoolAddressProvider, token0, token1, ammRouter, false));
    poolId = pools.length - 1;
    emit AddPool(poolId, lendingPoolAddressProvider);
}
```
This is due to a lack of existence check prior to pushing to the `pools` array which could have unintended consequences. Consider computing a unique `poolId` based on the input pool parameters so that it can be used for remedial existence checks, rather than simply using the zero-indexed number of elements in the array.