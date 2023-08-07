# Non-Critical issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Check pool validity before calling deprecatePool | 1 |
| [NC-2](#NC-2) | Check if pool is already deprecated or not | 1 |
| [NC-3](#NC-3) | Check if new variable is different from old | 1 |
| [NC-4](#NC-4) | Emit both old and new values after state change | 1 |
### <a name="NC-1"></a>[NC-1] Check pool validity before calling deprecatePool
`poolId` is the index of a pool in the `pools` array. We should check if the pool exists or not before passing the `poolId`.
We can check this by seeing if `lendingPoolAddressProvider` ,which is set whenever a new pool is created, is set or not.
We can add `require(pools[poolId].lendingPoolAddressProvider != address(0x0))` at the top of teh function to check this. 


Contracts in scope: https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol

*Instances (1)*:
```solidity
File: contracts/RoeRouter.sol

 L48:   function deprecatePool(uint poolId) public onlyOwner {
 
 L49:   pools[poolId].isDeprecated = true;
 
 L50:   emit DeprecatePool(poolId);
 
 L51:   }


```
### <a name="NC-2"></a>[NC-2] Check if pool is already deprecated or not
since the function emits an event `DeprecatePool`, any event listeners from third party apps can't be sure if this is the first time or not.
Hence, we should check if pool is already deprecated or not. we can add this line on top of the function
`require(!pools[poolId].isDeprecated)` 


Contracts in scope: https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol

*Instances (1)*:
```solidity
File: contracts/RoeRouter.sol

 L48:   function deprecatePool(uint poolId) public onlyOwner {
 
 L49:   pools[poolId].isDeprecated = true;
 
 L50:   emit DeprecatePool(poolId);
 
 L51:   }


```
### <a name="NC-3"></a>[NC-3] Check if new variable is different from old
When calling this function, we can add a check to make sure the new address is not the same as the old address.
We can put `require(newTreasury != treasury)` on top of the function.


Contracts in scope: https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol

*Instances (1)*:
```solidity
File: contracts/RoeRouter.sol

 L83:   function setTreasury(address newTreasury) public onlyOwner {
 
 L84:   require(newTreasury != address(0x0), "Invalid address");
 
 L85:   treasury = newTreasury;
 
 L86:   emit UpdateTreasury(newTreasury);

 L87:   }

```

### <a name="NC-4"></a>[NC-4] Check if new variable is different from old
This function only emits new value after state change. We can and should also emit the previous treasury address when emiting the `UpdateTreasury` event.


Contracts in scope: https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol

*Instances (1)*:
```solidity
File: contracts/RoeRouter.sol

 L83:   function setTreasury(address newTreasury) public onlyOwner {
 
 L84:   require(newTreasury != address(0x0), "Invalid address");
 
 L85:   treasury = newTreasury;
 
 L86:   emit UpdateTreasury(newTreasury);

 L87:   }

```