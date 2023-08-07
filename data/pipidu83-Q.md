
```Public``` functions not used in the contract itself should be marked as ```external``` 

This is a best practice advice and for the sake of clarity and gas usage will not be impacted with the solidity version used here (0.8.19).

We notice a few instances of this in the repository.

Instances:
GeVault.sol#L177-179

```solidity
function getTickLength() public view returns(uint len){
    len = ticks.length;
  }
```

RoeRouter.sol#L40-42

```solidity
function getPoolsLength() public view returns (uint poolLength) {
    poolLength = pools.length;
  }
```

RoeRouter.sol#L83-87

```solidity
function setTreasury(address newTreasury) public onlyOwner {
    require(newTreasury != address(0x0), "Invalid address");
    treasury = newTreasury;
    emit UpdateTreasury(newTreasury);
  }
```

TokenisableRange.sol#L361-363

```solidity
function latestAnswer() public view returns (uint256 priceX1e8) {
    return getValuePerLPAtPrice(ORACLE.getAssetPrice(address(TOKEN0.token)), ORACLE.getAssetPrice(address(TOKEN1.token)));
  }
```


