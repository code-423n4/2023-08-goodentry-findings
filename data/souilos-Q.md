# VULN 1 

## [LOW] Empty receive()/payable fallback() function does not authenticate requests
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 462 at 2023-08-good-entry/contracts/GeVault.sol:

  receive() external payable {


Found in line 86 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

    receive() external payable {

------------------------------------------------------------------------ 

### Mitigation 

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))).Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas. Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds. If the concern is having to spend a small amount of gas to check the sender against an immutable address, the code should at least have a function to rescue unused Ether.










# VULN 2 

## [LOW] Division before multiplication causing significant loss of precision
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 375 at 2023-08-good-entry/contracts/GeVault.sol:

    priceX8 = priceX8 / 10**decimals1;


Found in line 143 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    if (decimalsA > decimalsB) valueA = valueA / 10 ** (decimalsA - decimalsB);


Found in line 144 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

    else if (decimalsA < decimalsB) valueB = valueB / 10 ** (decimalsB - decimalsA);

------------------------------------------------------------------------ 

### Mitigation 

Multiply first before dividing to keep the precision.










# VULN 3 

## [LOW] Use .call instead of .transfer to send ether
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 232 at 2023-08-good-entry/contracts/GeVault.sol:

      payable(msg.sender).transfer(bal);

------------------------------------------------------------------------ 

### Mitigation 

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues.Replace .transfer with .call. Note that the result of .call need to be checked.










# VULN 4 

## [LOW] Use the safe variant and ERC721.mint
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 282 at 2023-08-good-entry/contracts/GeVault.sol:

    _mint(msg.sender, liquidity);    


Found in line 161 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    _mint(msg.sender, 1e18);


Found in line 281 at 2023-08-good-entry/contracts/TokenisableRange.sol:

    _mint(msg.sender, lpAmt);

------------------------------------------------------------------------ 

### Mitigation 

.mint won’t check if the recipient is able to receive the NFT. If an incorrect address is passed, it will result in a silent failure and loss of asset. OpenZeppelin recommendation is to use the safe variant of _mint. Replace _mint() with _safeMint().










# VULN 5 

## [LOW] Usage of deprecated chainlink API
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 289 at 2023-08-good-entry/contracts/GeVault.sol:

  function latestAnswer() external view returns (uint256 priceX8) {


Found in line 396 at 2023-08-good-entry/contracts/GeVault.sol:

      valueX8 += bal * t.latestAnswer() / 1e18;


Found in line 361 at 2023-08-good-entry/contracts/TokenisableRange.sol:

  function latestAnswer() public view returns (uint256 priceX1e8) {


Found in line 69 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

  function latestAnswer() external view returns (int256) {


Found in line 20 at 2023-08-good-entry/contracts/helper/FixedOracle.sol:

    function latestAnswer() external view returns (int256) {


Found in line 12 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    Note that only latestAnswer() is calculated, and this is primarily meant 


Found in line 50 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

  function latestAnswer() public view returns (int256) {


Found in line 79 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    return (type(uint80).max, latestAnswer(), block.timestamp, block.timestamp, type(uint80).max);


Found in line 341 at 2023-08-good-entry/contracts/PositionManager/OptionsPositionManager.sol:

    uint debtValue = TokenisableRange(debtAsset).latestAnswer() * debtAmount / 1e18;

------------------------------------------------------------------------ 

### Mitigation 

Use latestRoundData() instead of latestAnswer(). Also, adding checks for additional fields returned from latestRoundData() is recommended. https://docs.chain.link/data-feeds/price-feeds/api-reference/#latestrounddata 










# VULN 6 

## [LOW] safeApprove of openZeppelin is deprecated
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 116 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeApprove(address(ROUTER), amountIn);


Found in line 120 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeApprove(address(ROUTER), 0);


Found in line 128 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeApprove(address(ROUTER), amountInMax);


Found in line 133 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeApprove(address(ROUTER), 0);


Found in line 165 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeApprove(address(ROUTER), amountInMax);


Found in line 169 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeApprove(address(ROUTER), 0);


Found in line 183 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeApprove(address(ROUTER), amountIn);


Found in line 187 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

        ogInAsset.safeApprove(address(ROUTER), 0); 

------------------------------------------------------------------------ 

### Mitigation 

Deprecated, its usage is discouraged. https://docs.openzeppelin.com/contracts/3.x/api/token/erc20. Whenever possible, use safeIncreaseAllowance and safeDecreaseAllowance instead. https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#SafeERC20-safeIncreaseAllowance-contract-IERC20-address-uint256- & https://docs.openzeppelin.com/contracts/3.x/api/token/erc20#SafeERC20-safeDecreaseAllowance-contract-IERC20-address-uint256-










# VULN 7 

## [LOW] Named imports can be used
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 5 at 2023-08-good-entry/contracts/GeVault.sol:

import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";


Found in line 6 at 2023-08-good-entry/contracts/GeVault.sol:

import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";


Found in line 7 at 2023-08-good-entry/contracts/TokenisableRange.sol:

import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";


Found in line 8 at 2023-08-good-entry/contracts/TokenisableRange.sol:

import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";


Found in line 4 at 2023-08-good-entry/contracts/RangeManager.sol:

import "./openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";


Found in line 5 at 2023-08-good-entry/contracts/RangeManager.sol:

import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";


Found in line 7 at 2023-08-good-entry/contracts/RangeManager.sol:

import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";


Found in line 5 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

import "../openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";


Found in line 6 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

import "../openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";


Found in line 3 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

import "../openzeppelin-solidity/contracts/token/ERC20/ERC20.sol";


Found in line 4 at 2023-08-good-entry/contracts/PositionManager/PositionManager.sol:

import "../openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";

------------------------------------------------------------------------ 

### Mitigation 

It’s possible to name the imports to improve code readability. E.g. import “@openzeppelin/contracts/token/ERC20/IERC20.sol”; can be rewritten as import {IERC20} from `import `@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol` 










# VULN 8 

## [LOW] Immutables should be in uppercase
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 19 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

  AggregatorV3Interface public immutable CL_TOKENA;


Found in line 20 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

  AggregatorV3Interface public immutable CL_TOKENB;


Found in line 21 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

  UniswapV2Pair public immutable LP_TOKEN;


Found in line 22 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

	uint8 public immutable decimalsA;


Found in line 23 at 2023-08-good-entry/contracts/helper/LPOracle.sol:

	uint8 public immutable decimalsB;


Found in line 16 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    AggregatorV3Interface public immutable CL_TOKENA;


Found in line 17 at 2023-08-good-entry/contracts/helper/OracleConvert.sol:

    AggregatorV3Interface public immutable CL_TOKENB;


Found in line 62 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

    ISwapRouter immutable public ROUTER;


Found in line 63 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

    IQuoter     immutable public QUOTER;


Found in line 64 at 2023-08-good-entry/contracts/helper/V3Proxy.sol:

    uint24      immutable public feeTier; 

------------------------------------------------------------------------ 

### Mitigation 

Immutables should be in uppercase, it helps to distinguish immutables from other types of variables and provides better code readability.
