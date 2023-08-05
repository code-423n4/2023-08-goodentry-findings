### NC-1 Missing or Broken Links to Contracts. 
The audit scope [lists some contracts to be in scope](https://code4rena.com/contests/2023-08-good-entry#top). 

However these links in audit description are broken. It seems PoolAddress.sol and DataTypes.sol are libraries but they are placed in interfaces folder. 
- [IInitializableAToken.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/interfaces/IInitializableAToken.sol) 
- [DataTypes.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/interfaces/DataTypes.sol) 
- [PoolAddress.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/interfaces/PoolAddress.sol)

Impact:--> The above affects the auditability contracts as contracts may be thought to be needed, omitted or take time to find. 

Recommendation: --> Recommended links be corrected and files be moved to libs folder 

### NC-2 Inconsistent use of license Identifier - Worse is some contracts have no license identifier
Contracts in project are using using different SPDX license identifiers. Some of the contracts use **GPL-2.0-or-later**, others **none** others **agpl-3.0**

Here are the examples of the differences:
[OptionsPositionManager.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol) // SPDX-License-Identifier: none
[PositionManager.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol) // SPDX-License-Identifier: none
[FixedOracle.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/FixedOracle.sol#L1) blank missing license identifier 
[LPOracle.col](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol) // SPDX-License-Identifier: none
[V3Proxy.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/V3Proxy.sol#L1C1-L1C33) // SPDX-License-Identifier: UNLICENSED
[FixedPoint96.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/lib/FixedPoint96.sol#L1C1-L1C39) // SPDX-License-Identifier: GPL-2.0-or-later
[FullMath.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/lib/FullMath.sol#L1C1-L1C45) // SPDX-License-Identifier: GPL-3.0
[GetVault.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol) // SPDX-License-Identifier: UNLICENSED
[IAaveIncentivesController.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol) // SPDX-License-Identifier: agpl-3.0

Impact:--> The above leads to inconsistencies on the license for the project 

Recommendation: --> It is recommended to make use of same license identifier in allowed cases. Where not possible it is ideal to comment or document this 

### NC-3 Inconsistent use of _ (underscore) for function arguments. 
There are functions that make use of _ for function arguments and other functions do not use this, consider setter functions one uses it and another does not. In same function there are also cases where some arguments use _ and others do not. Inconsistencies are numerous in all contracts in scope to list so examples below used: 

[GetVault.sol line 67 ...constructor(  address _treasury, address roeRouter, address _uniswapPool, uint poolId, string memory name, string memory symbol,address weth,bool _baseTokenIsToken0)](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L67) 

[GetVault.sol line 101 ...function setEnabled(bool _isEnabled) ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L101) 

[GetVault.sol line 108 ...function setTreasury(address newTreasury) ](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L108)

Impact:--> The above leads to inconsistencies, affects code readability, maintanability 

Recommendation: --> It is recommended to make use of of consistency in applying _ to arguments and fix all cases in all contracts in scope 

### NC-4 Unused imports 

Some files have imports that are not used 

[GetVault.sol line 8 ...IAaveLendingPoolV2.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L8C23-L8C41)

[TokenisableRange.sol line 9.......Strings.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L9C49-L9C56)

[TokenisableRange.sol line 14 IAaveOracle has already been imported line 13.......Strings.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/TokenisableRange.sol#L14C23-L14C34)

[RangeManager.sol line 11 ...IAaveLendingPoolV2.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L11C23-L11C41)

[RangeManager.sol line 12 ...IAaveOracle.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L12C23-L12C35)

[RangeManager.sol line 13 ...IUniswapV2Pair.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L13C23-L13C37)

[RangeManager.sol line 14 ...IUniswapV2Factory.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L14C23-L14C40)

[RangeManager.sol line 15 ...IUniswapV2Router01.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L15C23-L15C41)

[RangeManager.sol line 16 ...ISwapRouter.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L16C23-L16C34)

[RangeManager.sol line 16 ...ISwapRouter.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L16C23-L16C34)

[RangeManager.sol line 16 ...IPriceOracle.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RangeManager.sol#L21C43-L21C55)

[PositionManager.sol line 8 ... IAaveLendingPoolV2.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/PositionManager.sol#L8C46-L8C64)

[PositionManager.sol line 11 ... IUniswapV2Pair.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/PositionManager.sol#L11C26-L11C40)

[PositionManager.sol line 12 ... IUniswapV2Factory.sol](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/PositionManager.sol#L12C26-L12C43)

Impact:--> The above leads to bloated code, code readability, code auditability, code maintanability issues as readers may think it implies missing code of functionality. 

**This doubles as a Gas Issue will not rereport it in Gas Report** Most importantly it adds to gas costs as it increases the size of the contract file as the unused library is extra bytecode for no reason which increases deployment costs for no reason  

Recommendation: --> It is recommended to remove all unused imports in above cases and any other relevant cases that may have been missed 

### NC-5 Inconsistent formatting and poor code formatting 

Opening code in VScode or use of other tool will show various format issues e. spacings, line starts, tabs, commas, etc. There are numerous to detail, the below are just examples

constructor starts too many tabs inside [OracleConvert.sol line 22](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/OracleConvert.sol#L22)
```
    AggregatorV3Interface public immutable CL_TOKENB;
	/// @param clToken0 Underlying token0 ChainLink feed
	/// @param clToken1 Underlying token1 ChainLink feed
	constructor (address clToken0, address clToken1 ){
    require(clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");
		CL_TOKENA = AggregatorV3Interface(clToken0);
		CL_TOKENB = AggregatorV3Interface(clToken1);
    require(CL_TOKENA.decimals() + CL_TOKENB.decimals() >= 16, "Decimals error");
	}
```

function below not very readable due to formatting [OracleConvert.sol line 37](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/helper/OracleConvert.sol#L37C3-L46C4)
```
function getAnswer(AggregatorV3Interface priceFeed) internal view returns (int256) {
    (
      , 
      int price,
      ,
      uint timeStamp,
    ) = priceFeed.latestRoundData();
    require(timeStamp > 0, "Round not complete");
    return price;
  }
```

Recommendation: --> It is recommended to run code files through a formatter and ensure code follows recommended Solidity style guides 

