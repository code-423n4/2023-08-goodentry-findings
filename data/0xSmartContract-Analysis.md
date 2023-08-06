# Analysis  - Good Entry 👍
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Architectural | Architecture feedback |
|e) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|f) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|g) |Systemic risks | Potential systemic risks in the project |
|h) |Competition analysis| What are similar projects? |
|i) |Security Approach of the Project | Audit approach of the Project |
|j) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|k) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|l) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-08-goodentry#scope

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-08-goodentry#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Good Entry Protocol](https://gitbook.goodentry.io/tokenomics) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither](https://github.com/crytic/slither)| |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-08-goodentry#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-08-goodentry#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-08-goodentry#scoping-details)||

## b) Analysis of the code base
The main most important contracts of the project;

### OptionsPositionManager:
(If you click on the Image below, you can see more detail in the larger image ):
<img width="1464" alt="image" src="https://github.com/code-423n4/2023-08-goodentry/assets/104318932/fbcce595-ba0d-473a-9a15-d55825076b30">

The use of slippage and deadline value in the architecture of the `swapExactTokensForTokens` function is not best practice. Leaving slippage and deadline value to be defined by the user is the best practice used in many projects;

```solidity
  function swapExactTokensForTokens(IUniswapV2Router01 ammRouter, IPriceOracle oracle, uint amount, address[] memory path) 
    internal returns (uint256 received)
  {
    if (amount > 0 && AmountsRouter(address(ammRouter)).getAmountsOut(amount, path)[1] > 0){
      checkSetAllowance(path[0], address(ammRouter), amount);
      uint[] memory amounts = ammRouter.swapExactTokensForTokens(
        amount, 
        getTargetAmountFromOracle(oracle, path[0], amount, path[1]) * 99 / 100, // allow 1% slippage  @audit-issue
        path, 
        address(this), 
        block.timestamp  // @audit-issue
      );
      received = amounts[1];
    }
  }
```

There are important functions in this contract and the most important operation of the project is here, in case of a potential problem, the project may need to be stopped, but Pause architecture was not used, I recommend adding Pause architecture
Functions that may need to be stopped with pause;

- executeOperation(address[],uint256[],uint256[],address,bytes)
- executeBuyOptions(uint256,address[],uint256[],address,address[])
- executeLiquidation(uint256,address[],uint256[],address,address)
- withdrawOptionAssets(uint256,address,uint256,address,address)
- buyOptions(uint256,address[],uint256[],address[])
- liquidate(uint256,address,address[],uint256[],address)
- withdrawOptions(uint256,address,uint256)
- swapTokens(uint256,address,uint256)
- swapExactTokensForTokens(IUniswapV2Router01,IPriceOracle,uint256,address[])


Another part is that there is no re-entrancy guard in swapped functions, re-entrancy guard should be added to these functions;


```solidity
contracts/PositionManager/OptionsPositionManager.sol:
  439:   function swapTokens(uint poolId, address sourceAsset, uint amount) public returns (uint received) {
  440:     (ILendingPool LP, IPriceOracle oracle,IUniswapV2Router01 router, address token0, address token1) = getPoolAddresses(poolId);
  441:     require(sourceAsset == token0 || sourceAsset == token1, "OPM: Invalid Swap Asset");
  442:     if (amount == 0) {
  443:       amount = ERC20(LP.getReserveData(sourceAsset).aTokenAddress).balanceOf(msg.sender);
  444:       if (amount == 0) return 0;
  445:     }
  446:     PMWithdraw(LP, msg.sender, sourceAsset, amount);
  447: 
  448:     address[] memory path = new address[](2);
  449:     path[0] = sourceAsset ;
  450:     path[1] = sourceAsset == token0 ? token1 : token0;
  451:     received = swapExactTokensForTokens(router, oracle, amount, path);
  452:     
  453:     cleanup(LP, msg.sender, token0);
  454:     cleanup(LP, msg.sender, token1);
  455:     emit Swap(msg.sender, path[0], amount, path[1], received);
  456:   }
```


### GeVault.sol:

<img width="1469" alt="image" src="https://github.com/code-423n4/2023-08-goodentry/assets/104318932/62e349ae-2189-45db-9cf1-236a6e3cc00a">


During `withdraw` function (withdraw function has reentrancy guard), they can run `rebalance` and this can cause reentrancy attack, so I suggest adding nonReentrancy modifier to `rebalance` function;
```solidity
contracts/GeVault.sol:
  202:   function rebalance() public {
  203:     require(poolMatchesOracle(), "GEV: Oracle Error");
  204:     removeFromAllTicks();
  205:     if (isEnabled) deployAssets();
  206:   }
```


Another question to consider is Oracle data health check for `withdraw` and `rebalance` functions to work, if Oracle data cannot be retrieved, functions are locked;
```solidity
contracts/GeVault.sol:
  202:   function rebalance() public {
  203:     require(poolMatchesOracle(), "GEV: Oracle Error");
  
  214:   function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {
  215:     require(poolMatchesOracle(), "GEV: Oracle Error");
```



####  Dependencies: 

It is observed that old and unsafe versions of Openzeppelin are used in the project, this should be updated

```solidity
contracts/openzeppelin-solidity/contracts/package.json:
  1: {
  2:   "name": "@openzeppelin/contracts",
  3:   "description": "Secure Smart Contract library for Solidity",
  4:   "version": "4.4.1",
```
https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/


-  Logic similar to singleton pattern similar to Uniswapv4 can be used in a project, this is a gas-optimized and innovative solution
- The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns;
https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1



## c) Test analysis

The audit scope of the contracts to be audited is 85% and it should be aimed to be 100%.

```
What is the overall line coverage percentage provided by your tests?: 85%
```

### What could they have done better?;
1 - There are many unit tests in the project, integration tests in which the interaction of contracts with each other are modeled should be increased.
The test scope is written in Python and the Brownie framework is used, I recommend that the tests are also written in Solidity with the Foundry framework,
Foundry is the smart contract development kit. It is used for the fast development of contracts, easy testing of smart contracts, and easy deployment of smart contracts.

Advantages of Foundry;
Writing tests in the first layer directly with Solidity models reality better
fast to develop
Built-in Fuzzing
Solidity based testing
EVM cheat codes
Scripts based in bash/shell

2- There are many unit tests in the project, integration tests in which the interaction of contracts with each other are modeled should be increased.

For example , this below codes for Chainlink `LatestRoundData `can be add in test suits

```solidity
pragma solidity ^0.8.0;

import "@forge-std/Test.sol";

import {MockChainlinkOracle} from "@test/mock/MockChainlinkOracle.sol";
import {ChainlinkUtils} from "LPOracle.sol";

contract ChainlinkUtilsIntegrationTest is Test {
    ChainlinkUtils public oracle;

    /// @notice multiplier value
    address public constant cbEthEthOracle =
        0xF017fcB346A1885194689bA23Eff2fE6fA5C483b;

    /// @notice eth usd value
    address public constant ethUsdOracle =
        0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419;

    function setUp() public {
        oracle = new ChainlinkUtils(ethUsdOracle, cbEthEthOracle, address(0));
    }

    function testSetup() public {
        assertEq(oracle.base(), ethUsdOracle);
        assertEq(oracle.multiplier(), cbEthEthOracle);
        assertEq(oracle.decimals(), 18);
    }

    function testcbETH_USD_CompositeOracle() public {
        uint256 price = oracle.getDerivedPrice(
            cbEthEthOracle,
            ethUsdOracle,
            18
        );
        assertTrue(price > 0, "Price should be greater than 0");
    }

    function testTestLatestRoundData() public {
        (
            uint80 roundId, /// always 0, value unused in ChainlinkOracle.sol
            int256 answer, /// the composite price
            uint256 startedAt, /// always 0, value unused in ChainlinkOracle.sol
            uint256 updatedAt, /// always block.timestamp
            uint80 answeredInRound /// always 0, value unused in ChainlinkOracle.sol
        ) = oracle.latestRoundData();
        uint256 price = oracle.getDerivedPrice(
            cbEthEthOracle,
            ethUsdOracle,
            18
        );

        assertTrue(answer > 0, "Price should be greater than 0");

        assertEq(price, uint256(answer));
        assertEq(updatedAt, block.timestamp);
        assertEq(roundId, 0);
        assertEq(startedAt, 0);
        assertEq(answeredInRound, 0);
    }
}
```


### Why  integration tests is important?
![image](https://github.com/code-423n4/2023-07-tapioca/assets/104318932/109aa8ae-5754-4c5c-9244-3c22a7b96560)


### Test suites do not test for attack vectors, especially re-entrancy

Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here.
https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability

Attack vectors, especially re-entrancy, seem untested and trusted

```solidity


contracts/GeVault.sol:
  214:   function withdraw(uint liquidity, address token) public nonReentrant returns (uint amount) {
  247:   function deposit(address token, uint amount) public payable nonReentrant returns (uint liquidity) 

contracts/RangeManager.sol:
  139:   function removeAssetsFromStep(uint256 step) nonReentrant external {
  175:   function transferAssetsIntoRangerStep(uint256 step, uint256 amount0, uint256 amount1) nonReentrant external {
  184:   function transferAssetsIntoTickerStep(uint256 step, uint256 amount0, uint256 amount1) nonReentrant external {

contracts/TokenisableRange.sol:
  222:   function deposit(uint256 n0, uint256 n1) external nonReentrant returns (uint256 lpAmt) {
  292:   function withdraw(uint256 lp, uint256 amount0Min, uint256 amount1Min) external nonReentrant returns (uint256 removed0, uint256 removed1) {


```

## d) Architectural 

<img width="800" alt="image" src="https://github.com/code-423n4/2023-08-goodentry/assets/104318932/0080a795-a25d-4330-9e82-73d43c82f46f">

<img width="800" alt="image" src="https://github.com/code-423n4/2023-08-goodentry/assets/104318932/948f9fcb-0f1b-4da7-a0ff-36f1117053c5">




## e) Documents 
- Documentation should be increased further, it is recommended to add the architectural design to the documents as infographic

- I would also recommend adding quality Medium articles, it's a great way to provide an in-depth look at many of the topics in the project and is used by many blockchain projects.

##  f) Centralization risks 
There is a risk of centrality in the project as the onlyOwner
The owner role has a single point of failure and onlyOwner can use critical functions, posing a centralization issue. There is always a chance for owner keys to be stolen, and in such a case, the attacker can cause serious damage to the project due to important functions.

The code detail of this topic is specified in automatic finding;
https://gist.github.com/thebrittfactor/691192755086c4aa311b69a9de0690f3#medium-2-privileged-functions-can-create-points-of-failure

```solidity
contracts/GeVault.sol:
  101:   function setEnabled(bool _isEnabled) public onlyOwner { 
  108:   function setTreasury(address newTreasury) public onlyOwner { 
  116:   function pushTick(address tr) public onlyOwner {
  137:   function shiftTick(address tr) public onlyOwner {
  167:   function modifyTick(address tr, uint index) public onlyOwner {
  183:   function setBaseFee(uint newBaseFeeX4) public onlyOwner {
  191:   function setTvlCap(uint newTvlCap) public onlyOwner {

contracts/RangeManager.sol:
  75:   function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {
  95:   function initRange(address tr, uint amount0, uint amount1) external onlyOwner {

contracts/RoeRouter.sol:
  48:   function deprecatePool(uint poolId) public onlyOwner {
  65:     public onlyOwner 
  83:   function setTreasury(address newTreasury) public onlyOwner {

contracts/helper/FixedOracle.sol:
  10:     modifier onlyOwner() {
  24:     function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {

contracts/helper/V3Proxy.sol:
  94:     function emergencyWithdraw(ERC20 token) onlyOwner external {   

```

Reducing the onlyOwner centrality risk in blockchain projects is essential to enhance security, decentralization, and community trust. Relying too heavily on a single owner or administrator can lead to various issues, such as single points of failure, potential abuse of power, and lack of transparency. Here are some strategies to mitigate the onlyOwner centrality risk:

1-Multi-Signature (Multi-Sig) Wallets: Instead of having a single owner with full control, you can implement multi-signature wallets. Multi-sig wallets require multiple parties to sign off on transactions or changes to the smart contract, making it more decentralized and secure.

2-Timelocks and Governance Delays: Introduce timelocks or governance delays for critical operations or updates. This means that proposed changes need to wait for a specified period before being executed. During this period, the community can review and potentially veto the changes if they identify any issues.

3-Decentralized Governance Mechanisms: Implement decentralized governance mechanisms, such as DAOs (Decentralized Autonomous Organizations) or community voting systems. This allows token holders or stakeholders to have a say in important decisions, making the project more community-driven.

Great summary of GalloDaSballo with examples of centrality risk and how to reduce it in previous Code4rena audits;
https://gist.github.com/GalloDaSballo/881e7a45ac14481519fb88f34fdb8837

##  g) Systemic risks 
The most important system risk in the project; Risks arising from the use of Oracle, the project uses Chainlink, oracles, security exploits from oracles have been increasing recently, so using Oracle is a risk in itself and it is important to minimize this risk.

We can talk about a systemic risk here because there are some Layer2 special cases.
Some conventions in the project are set to Pragma ^0.8.19, allowing the conventions to be compiled with any 0.8.x compiler. The problem with this is that Arbitrum is [Compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer. Contracts compiled with these versions will result in a non-functional or potentially damaged version that does not behave as expected. The default behavior of the compiler will be to use the latest version which means it will compile with version 0.8.20 which will produce broken code by default.


## h) Competition analysis

Derivatives are financial contracts between two or more parties that derive their value from the price of an underlying asset or index (“the underlying”). Derivatives may reference various underlying assets like stocks, interest rates, commodities, currencies, crypto, and/or others

In addition to the volume from centralized exchanges and OTC desks, a small portion of options volume occurs onchain via decentralized option protocols. Option DEXs allow users to mint and trade vanilla options onchain, and they typically trade using an AMM, a liquidity pool with a pricing oracle, or an order book. The majority of activity has congregated on AMM/liquidity pool-based option DEXs like Lyra and Dopex so far. With order books requiring greater transaction throughput, the majority of these protocols were built on Solana and have seen diminished activity in the wake of FTX. Despite Ethereum’s Opyn v2 deprecating its 0x-powered onchain order book, Opyn is still a top option protocol by TVL due to its use as an infrastructure layer to mint options onchain.

<img width="734" alt="image" src="https://github.com/code-423n4/2023-08-goodentry/assets/104318932/936dcf47-0ea6-4da9-8e99-a462074b47e0">

Though we are still in the early stages of derivative DEX development, several approaches are used to facilitate the decentralized trading of options and perps via smart contracts. 

<img width="540" alt="image" src="https://github.com/code-423n4/2023-08-goodentry/assets/104318932/6bfbaa88-8e4f-41c3-9f6f-02e9f9fbea73">



## i) Security Approach of the Project

Successful current security understanding of the project;
1- The first audit was conducted in a reputable audit firm such as PeckShield and all security concerns were resolved, this is the case in both audit reports;

https://gitbook.goodentry.io/audits

2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.



What the project should add in the understanding of Security;
1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)
2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)
3- After the inspection, the concept of "continuous inspection" should be adhered to with bug bounty programs such as ImmuneFi.


## j) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://gist.github.com/thebrittfactor/691192755086c4aa311b69a9de0690f3

Especially High and Medium detections in the Automated Finding Report should be taken into account;

# Summary for HIGH findings

| Number | Details | Instances |
|----------|----------|----------|
| [HIGH-2] | Dangerous arithmetic using unvalidated latestRoundData int price return | 6 |
# Summary for MEDIUM findings

| Number      | Details | Instances |
|-------------|----------|----------|
| [MEDIUM-1]  | SafeTransfer should be used in place of transfer | 3 |
| [MEDIUM-2]  | Privileged functions can create points of failure | 13 |
| [MEDIUM-3]  | safeMint should be used in place of mint | 3 |
| [MEDIUM-4]  | Contracts are vulnerable to fee-on-transfer accounting-related issues | 6 |
| [MEDIUM-5]  | No refunds for swaps  | 3 |
| [MEDIUM-6]  | Return values of transfer()/transferFrom() not checked | 5 |
| [MEDIUM-7]  | Decimals() underflow | 1 |
| [MEDIUM-8]  | Withdraw on tokens which are payable but have no withdraw functionality | 2 |
| [MEDIUM-9]  | Chainlink price feeds are not validated | 2 |
| [MEDIUM-10] | Greater than comparisons made on state uints that can be set to max | 2 |
| [MEDIUM-11] | Use call instead of transfer on payable addresses | 2 |
| [MEDIUM-12] | Missing checks for whether the L2 Sequencer is active | 2 |
| [MEDIUM-13] | Use lastestRoundData in place of latestAmount | 8 |
| [MEDIUM-14] | Remaining eth may not be refunded to users | 1 |
| [MEDIUM-15] | Insufficient oracle validation | 1 |

**Other Audit Reports:**
https://github.com/GoodEntry-io/audits/blob/main/PeckShield-Audit-Report-GoodEntry-v1.0.pdf

https://github.com/GoodEntry-io/audits/blob/main/Roe-Protofire-Audit.pdf

## k) Gas Optimization

The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size

Highest gas optimization: It is Storage Packed, high gas gain will be achieved in case of packaging in the following structs


https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ISwapRouter.sol#L10-L17
```javascript
10:     struct ExactInputSingleParams {
11:         address tokenIn;
12:         address tokenOut;
13:         uint24 fee;
14:         address recipient;
15:         uint256 deadline; // <= Storage Packed
16:         uint256 amountIn; // <= Storage Packed
17:         uint256 amountOutMinimum; // <= Storage Packed
18:         uint160 sqrtPriceLimitX96;
19:     }

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ISwapRouter.sol#L39-L46
```javascript
39:     struct ExactOutputSingleParams {
40:         address tokenIn;
41:         address tokenOut;
42:         uint24 fee;
43:         address recipient;
44:         uint256 deadline; // <= Storage Packed
45:         uint256 amountOut; // <= Storage Packed
46:         uint256 amountInMaximum; // <= Storage Packed
47:         uint160 sqrtPriceLimitX96;
48:     }

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L79-L90
```javascript
79:     struct MintParams {
80:         address token0;
81:         address token1;
82:         uint24 fee;
83:         int24 tickLower;
84:         int24 tickUpper;
85:         uint256 amount0Desired; // <= Storage Packed
86:         uint256 amount1Desired; // <= Storage Packed
87:         uint256 amount0Min; // <= Storage Packed
88:         uint256 amount1Min; // <= Storage Packed
89:         address recipient;
90:         uint256 deadline; // <= Storage Packed
91:     }

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L111-L117
```javascript
111:     struct IncreaseLiquidityParams {
112:         uint256 tokenId; // <= Storage Packed
113:         uint256 amount0Desired; // <= Storage Packed
114:         uint256 amount1Desired; // <= Storage Packed
115:         uint256 amount0Min; // <= Storage Packed
116:         uint256 amount1Min; // <= Storage Packed
117:         uint256 deadline; // <= Storage Packed
118:     }

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L139-L144
```javascript
139:     struct DecreaseLiquidityParams {
140:         uint256 tokenId; // <= Storage Packed
141:         uint128 liquidity;
142:         uint256 amount0Min; // <= Storage Packed
143:         uint256 amount1Min; // <= Storage Packed
144:         uint256 deadline; // <= Storage Packed
145:     }

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ISwapRouter.sol#L26-L31
```javascript
26:     struct ExactInputParams {
27:         bytes path;
28:         address recipient;
29:         uint256 deadline; // <= Storage Packed
30:         uint256 amountIn; // <= Storage Packed
31:         uint256 amountOutMinimum; // <= Storage Packed
32:     }

```

https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ISwapRouter.sol#L55-L60
```javascript
55:     struct ExactOutputParams {
56:         bytes path;
57:         address recipient;
58:         uint256 deadline; // <= Storage Packed
59:         uint256 amountOut; // <= Storage Packed
60:         uint256 amountInMaximum; // <= Storage Packed
61:     }

```
The variables mentioned above can be packaged in several ways; The number of storage slots can be reduced by changing the order and decreasing the uint value.


## l) New insights and learning from this audit 


- I discovered how LP Derivatives Markets are embedded in smart codes
- Learned how to use Chainlink Oracles to prevent price manipulation and set rebalancing thresholds
- I learned the Protected Perps algorithm and how it is coded
- I learned the methods of optimizing yield production
- I learned the `aave-compatible flashloan receiver dispatch` design, an interesting architecture that I haven't come across before


### Time spent:
17 hours