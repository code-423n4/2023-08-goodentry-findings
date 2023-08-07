# Description
The following source units are imported but not referenced in the contract:

**PositionManager.sol:**
```solidity
import "../../interfaces/IUniswapV2Pair.sol";
import "../../interfaces/IUniswapV2Factory.sol";
```

**GeVault.sol:**
```solidity
import "../interfaces/IAaveLendingPoolV2.sol";
```

**RangeManager.sol:**
```solidity
import "./openzeppelin-solidity/contracts/utils/cryptography/ECDSA.sol";
import "../interfaces/AggregatorV3Interface.sol";
import "../interfaces/IAaveLendingPoolV2.sol";
import "../interfaces/IUniswapV2Pair.sol";
import "../interfaces/IUniswapV2Factory.sol";
import "../interfaces/IUniswapV2Router01.sol";
import "../interfaces/ISwapRouter.sol";
import {IPriceOracle} from "../interfaces/IPriceOracle.sol";
```

**TokenisableRange.sol:**
```solidity
import "./openzeppelin-solidity/contracts/utils/Strings.sol";
```

# Recommended Mitigation Steps
Consider removing this imports.