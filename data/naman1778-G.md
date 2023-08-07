## [G-01] Functions guaranteed to revert when called by normal users can be marked *payable*

If a function modifier such as *onlyOwner* is used, the function will revert if a normal user tries to pay the function. Marking the function as *payable* will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are *CALLVALUE*(2),*DUP1*(3),*ISZERO*(3),*PUSH2*(3),*JUMPI*(10),*PUSH1*(3),*DUP1*(3),*REVERT*(0),*JUMPDEST*(1),*POP*(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

There are 14 instances of this issue in 5 files: 

    File: GeVault.sol	

    101: function setEnabled(bool _isEnabled) public onlyOwner { 

    108: function setTreasury(address newTreasury) public onlyOwner { 

    116: function pushTick(address tr) public onlyOwner {

    137: function shiftTick(address tr) public onlyOwner {

    167: function modifyTick(address tr, uint index) public onlyOwner {

    183: function setBaseFee(uint newBaseFeeX4) public onlyOwner {

    191: function setTvlCap(uint newTvlCap) public onlyOwner {

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

    File: V3Proxy.sol	

    94: function emergencyWithdraw(ERC20 token) onlyOwner external {   

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol

    File: RangeManager.sol	

    75: function generateRange(uint128 startX10, uint128 endX10, string memory startName, string memory endName, address beacon) external onlyOwner {  

    95: function initRange(address tr, uint amount0, uint amount1) external onlyOwner {

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol

    File: RoeRouter.sol	

    48: function deprecatePool(uint poolId) public onlyOwner {

    59: function addPool(
    60:   address lendingPoolAddressProvider, 
    61:   address token0, 
    62:   address token1, 
    63:   address ammRouter
    64: ) 
    65:   public onlyOwner 
    66:   returns (uint poolId)

    83: function setTreasury(address newTreasury) public onlyOwner {

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol

    File: FixedOracle.sol	

    24: function setHardcodedPrice(int256 _hardcodedPrice) external onlyOwner {

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    
    contract Contract0 {
        address owner = msg.sender;
        uint256 num;
    
        modifier only_owner{
             require(msg.sender==owner);
             _;
        }
    
        function not_optimized() public only_owner {
             num = 1;
        }
    }
    
    contract Contract1 {
    
        address owner = msg.sender;
        uint256 num;
    
        modifier only_owner{
             require(msg.sender==owner);
             _;
        }
    
        function optimized() public only_owner payable {
             num = 1;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 51816                                     | 196             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24356           | 24356 | 24356  | 24356 | 1       |

| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 49416                                     | 184             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24332           | 24332 | 24332  | 24332 | 1       |

## [G-02] Use assembly to write address storage values

There are 6 instances of this issue in 4 files: 

    File: GeVault.sol	

    89: treasury = _treasury;

    109: treasury = newTreasury; 

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

    File: TokenisableRange.sol	

    88: creator = msg.sender;

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol

    File: RoeRouter.sol	

    35: treasury = treasury_;

    85: treasury = newTreasury;

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol

    File: FixedOracle.sol	

    16: owner = msg.sender;

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/FixedOracle.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.setAddress(0x0000000000000000000000000000000000000001);
            c1.setAddressOptimized(0x0000000000000000000000000000000000000001);
        }
    }
    contract Contract0 {
        address a;
        function setAddress(address _a) public returns(bytes32){
            a = _a;
        }
    }
    contract Contract1 {
        address a;
        function setAddressOptimized(address _a) public returns(bytes32){
            assembly
            {
                sstore(a.slot, _a)
            }
        }
    }



#### Gas Report

| src/test/GasTest.t.sol:Contract0 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 51905                                     | 291             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setAddress                                | 22411           | 22411 | 22411  | 22411 | 1       |

| src/test/GasTest.t.sol:Contract1 contract |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 39093                                     | 226             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setAddressOptimized                       | 22378           | 22378 | 22378  | 22378 | 1       |

## [G-03] Using XOR (*^*) and OR (*|*) bitwise equivalents

On Remix, given only uint256 types, the following are logical equivalents, but don’t cost the same amount of gas:

(a != b || c != d || e != f) costs 571
((a ^ b) | (c ^ d) | (e ^ f)) != 0 costs 498 (saving 73 gas)
Consider rewriting as following to save gas:

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.
Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.

Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas:

However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values

There are 4 instances of this issue in 2 files: 

    File: OptionsPositionManager.sol	

    137: require(sourceSwap == token0 || sourceSwap == token1, "OPM: Invalid Swap Token");

    261: require(collateralAsset == token0 || collateralAsset == token1, "OPM: Invalid Collateral Asset");

    441: require(sourceAsset == token0 || sourceAsset == token1, "OPM: Invalid Swap Asset");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol

    File: GeVault.sol	

    251: require(token == address(token0) || token == address(token1), "GEV: Invalid Token");

    274: if (tSupply == 0 || vaultValueX8 == 0)

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }

    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }

    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |

## [G-04] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend

There are 5 instances of this issue in 2 files: 

    File: OptionsPositionManager.sol	

    45: uint8 mode = abi.decode(params, (uint8) );

    168: bytes memory params = abi.encode(0, poolId, msg.sender, sourceSwap);

    199:  bytes memory params = abi.encode(1, poolId, user, collateralAsset); // mode = 1 -> liquidation

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol

    File: GeVault.sol	

    394: TokenisableRange t = ticks[k];

    421: TokenisableRange t = ticks[index];

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

#### Test Code 

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    
    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }
    
    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |

| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |

## [G-05] Splitting *require()* statements that use *&&* saves gas

See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by 3 gas.

There are 14 instances of this issue in 7 files: 

    File: OptionsPositionManager.sol	

    167: require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");

    495: require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );

    523: require ( priceAssetA > 0 && priceAssetB > 0, "OPM: Invalid Oracle Price");

    536: require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol

    File: GeVault.sol	

    120: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

    141: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

    170: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

    File: TokenisableRange.sol	

    209: require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");

    271: require (TOKEN0_PRICE > 0 && TOKEN1_PRICE > 0, "Invalid Oracle Price");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol

    File: RangeManager.sol	

    108: require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol

    File: LPOracle.sol	

    29: require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");

    101: require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol

    File: RoeRouter.sol	

    68: require (
    69:   lendingPoolAddressProvider != address(0x0) 
    70:   && token0 != address(0x0) 
    71:   && token1 != address(0x0) 
    72:   && ammRouter != address(0x0), 
    73:   "Invalid Address"
    74: );

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol

    File: OracleConvert.sol	

    23: require(clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/OracleConvert.sol/

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }

    contract Contract0 {

        function checkAge(uint8 _age) public returns(string memory){
            require(_age>18 && _age<22);
            return "Eligible";
        }

    }

    contract Contract1 {

        function checkAgeOptimized(uint8 _age) public returns(string memory){
            require(_age>18);
            require(_age<22);
            return "Eligible";
        }
    }

#### Gas Report 

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76723                                     | 415             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 649             | 649 | 649    | 649 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 641             | 641 | 641    | 641 | 1       |

## [G-06] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

There are 10 instances of this issue in 3 files: 

    File: OptionsPositionManager.sol	

    71: for ( uint8 k = 0; k<assets.length; k++){

    93: for ( uint8 k =0; k<assets.length; k++){

    172: for (uint8 i = 0; i< options.length; ){
    174:   unchecked { i+=1; }

    203: for (uint8 i = 0; i< options.length; ){
    205:   unchecked { i+=1; }

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol

    File: GeVault.sol	

    154: for (uint k = 0; k < ticks.length - 2; k++)

    299: for (uint k = 0; k < ticks.length; k++){

    314: for (uint k = 0; k < ticks.length; k++){

    393: for(uint k=0; k<ticks.length; k++){

    431: for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

    File: RangeManager.sol	

    62: for (uint i = 0; i < len; i++) { 

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function not_optimized() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }


    contract Contract1 {
        uint256 num = 3;
        function optimized() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 7040            | 7040 | 7040   | 7040 | 1       |
| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 3819            | 3819 | 3819   | 3819 | 1       |

## [G-07] Using *private* rather than *public* for *constants*, saves gas

If needed, the value can be read from the verified contract source code. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table

There are 6 instances of this issue in 3 files: 

    File: GeVault.sol	

    62: uint public constant nearbyRanges = 2;

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

    File: TokenisableRange.sol	

    58: INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88); 
    
    59: IUniswapV3Factory constant public V3_FACTORY = IUniswapV3Factory(0x1F98431c8aD98523631AE4a59f267346ea31F984);
     
    60: address constant public treasury = 0x22Cc3f665ba4C898226353B672c5123c58751692;
    
    61: uint constant public treasuryFee = 20;

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol

    File: RangeManager.sol	

    36: INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);  

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol
