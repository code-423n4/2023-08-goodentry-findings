`Dear Good Entry team, as we have gone through each contract within the scope, we have noticed very good practices that have been implemented. However, we have identified some inconsistencies that we recommend addressing.We believe that at least 90% of the recommendations in the following report should be applied for better readability, and most importantly, safety.`

`Note: We have provided a description of the situation and recommendations to follow, including articles and resources we have created to help identify the problem and address it quickly, and to implement them in future standasrs.`

# QA Report for Good Entry contest

## [01] If onlyOwner runs renounceOwnership() the contract become unavailable
The `onlyOwner` authority here is very important for the contracts, but the `Ownable.sol library` imported with the import has the renounceOwnership() feature, in case the owner accidentally triggers this function, the remove functions will not work and the contract will block gas due to arrays, may have a continuous structure that exceeds its limit.

### Proof of Concept
- [GeVault.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol#L4)
- [RangeManager.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol#L20)
- [RoeRouter.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol#L4)

The solution to this is to overide and disable the renounceOwnership() function as implemented in many contracts in his project, it is important to include this code in the contracts.

```solidity
function renounceOwnership() public payable override onlyOwner {
    revert("Cannot renounce ownership");
}
```

## [02] You should check an important input in the function GeVault.setTreasury()
This function GeVault.setTreasury dont check newTreasury != address(0). 

### Proof of Concept
```solidity
contracts/GeVault.sol#L108-L111
 function setTreasury(address newTreasury) public onlyOwner {
    // @audit add this check
    require(newTreasury != address(0), "invalid address");  <------ 
    treasury = newTreasury; 
    emit SetTreasury(newTreasury);
  }
```
## [03] Crucial information is missing on important functions in all contracts
In some functions it is not specified in the documentation if the onlyOwner will be an EOA or a contract. Consider improving the docstrings to reflect the exact intended behaviour.

## [04] GeVault.setTvlCap() should add one more parameter in the event
GeVault.setTvlCap() add one more parameter in the event to be able to continue tracing on chain has been its previous TvlCap and same happens with GeVault.setBaseFee().

## [05] It seems that the max base fee x4 will be 1e4
In the require statement inside the GeVault.setBaseFee() function, there is a check specifying that the newBaseFee should be less than 1e4. There should be a constant variable defining maxBaseFeeX4, so that auditors or individuals who want to reference this in the source code are aware of it. When we asked the sponsor about our doubts in 1e4 via DM, they acknowledged that we were right.

### Proof of Concept
```solidity
contracts/GeVault.sol
// @audit how could look the change
  function setBaseFee(uint newBaseFeeX4) public onlyOwner {
  require(newBaseFeeX4 < maxBaseFeeX4, "GEV: Invalid Base Fee");
    baseFeeX4 = newBaseFeeX4;
    emit SetFee(newBaseFeeX4);
  }
```
NicoDeva on DM:
```text
0xcatellatech — hoy a las 19:57
The Max Base Fee X4 it will be 1e4?

NicoDeva — hoy a las 20:46
yes X4 means e4, it's the same in chainlink or Aave contracts
```

## [06] In GeVault.receive() the msg.value have to be > 0
Add some check in this function that define the msg.value have to greater than 0.

## [07] Naming issue in GeVault.sol
In the GeVault.sol contract we recommend refactoring the variable name `lpap` in the constructor in order to greatly benefit development and code understanding by the reviewer.

### Proof of Concept
`lpap` should be renamed to `lendingPoolAddProvider` to better represent what it does.

```solidity
contracts/GeVault.sol
// @audit refactorize the name with --> lendingPoolAddProvider
83: (address lpap, address _token0, address _token1,, )...
87: lendingPool = ILendingPool(ILendingPoolAddressesProvider(lpap).getLendingPool());
88: oracle = IPriceOracle(ILendingPoolAddressesProvider(lpap).getPriceOracle());
```

## [08] Avoid having comments that are unrelated to the code
In the GeVault contract, lines 57-58 contain comments that are unrelated to the contract's logic. Please remove these lines of code to improve readability. 

### Proof of Concept
```solidity
contracts/GeVault.sol#L56-L57
@audit Remove this lines to improve readability
/// CONSTANTS 
/// immutable keyword removed for coverage testing bug in brownie
```

## [09] Array does not have a pop function
Arrays without the pop operation in Solidity can lead to inefficient memory management and increase the likelihood of out-of-gas errors.

### Proof of Concept
```solidity
contracts/GeVault.sol 
121: if (ticks.length == 0) ticks.push(t); 
129: ticks.push(TokenisableRange(tr));
142: if (ticks.length == 0) ticks.push(t);
151: ticks.push(ticks[ticks.length-1]);
```
## [10] In some contracts, we find inconsistencies regarding uint256
In some contracts use uint instead uint256, some developers prefer to use uint256 because it is consistent with other uint data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs. insecure and more error-prone. 

## [11] In some contracts variable names should be in uppercase
We recommend maintaining consistency in the code and naming constants variables in uppercase. However, in the TokenisableRange contract, some variables follow this convention while others do not. It is recommended to be consistent with your own practices.

### Proof of Concept
```solidity
TokenisableRange.sol#L58-L61
58: INonfungiblePositionManager constant public POS_MGR.. 
59: IUniswapV3Factory constant public V3_FACTORY = IUniswapV3Factory... 
60: address constant public treasury = 0x22Cc3f665ba4C898226353B672c5123c58751692;
61: uint constant public treasuryFee = 20;
```

### Recommendation
It is important to adhere to good Solidity coding practices to maintain code clarity and project maintainability in both the short and long term. Consistency in following these practices ensures that the code remains readable and understandable. This promotes better collaboration among developers and reduces the chances of introducing errors or confusion in the codebase.

## [12] We suggest to use named parameters for mapping type
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.

```solidity
mapping(address account => uint256 balance); 
```

## [13] Use a single file for all system-wide constants
In some contracts, there are many "constants" used in the system. It is recommended to store all constants in a single file (for example, Constants.sol) and use inheritance to access these values.

This helps improve readability and facilitates future maintenance. It also helps with any issues, as some of these hard-coded contracts are administrative contracts.

Constants.sol is used and imported in contracts that require access to these values. This is just a suggestion.