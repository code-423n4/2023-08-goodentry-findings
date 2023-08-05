## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `abi.encode()` is less efficient than `abi.encodepacked()` | 2 | 106 |
| [GAS&#x2011;2](#GAS&#x2011;2) | `<array>.length` Should Not Be Looked Up In Every Loop Of A For-loop | 3 | 291 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Use assembly in place of `abi.decode` to extract calldata values more efficiently | 2 | 224 |
| [GAS&#x2011;4](#GAS&#x2011;4) | Use assembly to emit events | 35 | 1330 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Avoid emitting event on every iteration | 1 | 375 |
| [GAS&#x2011;6](#GAS&#x2011;6) | Counting down in `for` statements is more gas efficient | 9 | 2313 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function | 24 | 672 |
| [GAS&#x2011;8](#GAS&#x2011;8) | Multiple accesses of a mapping/array should use a local variable cache | 21 | 1680 |
| [GAS&#x2011;9](#GAS&#x2011;9) | The result of a function call should be cached rather than re-calling the function | 20 | 1000 |
| [GAS&#x2011;10](#GAS&#x2011;10) | Splitting `require()` Statements That Use `&&` Saves Gas | 10 | 90 |
| [GAS&#x2011;11](#GAS&#x2011;11) | Structs can be packed into fewer storage slots | 1 | 2000 |
| [GAS&#x2011;12](#GAS&#x2011;12) | Use do while loops instead of for loops | 9 | 36 |
| [GAS&#x2011;13](#GAS&#x2011;13) | Use nested `if` and avoid multiple check combinations | 8 | 48 |
| [GAS&#x2011;14](#GAS&#x2011;14) | Using XOR (^) and AND (&) bitwise equivalents | 48 | 624 |

Total: 227 contexts over 14 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
File: OptionsPositionManager.sol

168: bytes memory params = abi.encode(0, poolId, msg.sender, sourceSwap);
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L168

```solidity
File: OptionsPositionManager.sol

199: bytes memory params = abi.encode(1, poolId, user, collateralAsset);
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L199



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
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |



### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> `<array>.length` Should Not Be Looked Up In Every Loop Of A For-loop

The overheads outlined below are PER LOOP, excluding the first loop

storage arrays incur a Gwarmaccess (100 gas)
memory arrays use MLOAD (3 gas)
calldata arrays use CALLDATALOAD (3 gas)

Caching the length changes each of these to a DUP<N> (3 gas), and gets rid of the extra DUP<N> needed to store the stack offset

These instances include instances that haven't been detected in the automated findings.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

314: for (uint k = 0; k < ticks.length; k++){
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L314

```solidity
File: GeVault.sol

393: for(uint k=0; k<ticks.length; k++){
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L393

```solidity
File: OptionsPositionManager.sol

203: for (uint8 i = 0; i< options.length; ){
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L203

</details>




### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Use assembly in place of `abi.decode` to extract calldata values more efficiently

Instead of using `abi.decode`, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

For example, for a generic `abi.decode` call:
```solidity
(bytes32 varA, bytes32 varB, uint256 varC, uint256 varD) = abi.decode(metadata, (bytes32, bytes32, uint256, uint256));
```

We can use the following assembly call to extract the relevant metadata:

```solidity
        bytes32 varA;
        bytes32 varB;
        uint256 varC;
        uint256 varD;
        { // used to discard `data` variable and avoid extra stack manipulation
            bytes calldata data = metadata;
            assembly {
                varA := calldataload(add(data.offset, 0x00))
                varB := calldataload(add(data.offset, 0x20))
                varC := calldataload(add(data.offset, 0x40))
                varD := calldataload(add(data.offset, 0x60))
            }
        }
```

#### <ins>Proof Of Concept</ins>

```solidity
File: OptionsPositionManager.sol

48: (, uint poolId, address user, address[] memory sourceSwap) = abi.decode(params, (uint8, uint, address, address[]));
53: (, uint poolId, address user, address collateral) = abi.decode(params, (uint8, uint, address, address));

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L48

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L53








### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Use assembly to emit events

We can use assembly to emit events efficiently by utilizing `scratch space` and the `free memory pointer`. This will allow us to potentially avoid memory expansion costs.
Note: In order to do this optimization safely, we will need to cache and restore the free memory pointer.

For example, for a generic `emit` event for `eventSentAmountExample`:
```solidity
// uint256 id, uint256 value, uint256 amount
emit eventSentAmountExample(id, value, amount);
```

We can use the following assembly emit events:

```solidity
        assembly {
            let memptr := mload(0x40)
            mstore(0x00, calldataload(0x44))
            mstore(0x20, calldataload(0xa4))
            mstore(0x40, amount)
            log1(
                0x00,
                0x60,
                // keccak256("eventSentAmountExample(uint256,uint256,uint256)")
                0xa622cf392588fbf2cd020ff96b2f4ebd9c76d7a4bc7f3e6b2f18012312e76bc3
            )
            mstore(0x40, memptr)
        }
```

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

103: emit SetEnabled(_isEnabled);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L103

```solidity
File: GeVault.sol

110: emit SetTreasury(newTreasury);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L110

```solidity
File: GeVault.sol

131: emit PushTick(tr);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L131

```solidity
File: GeVault.sol

160: emit ShiftTick(tr);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L160

```solidity
File: GeVault.sol

172: emit ModifyTick(tr, index);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L172

```solidity
File: GeVault.sol

186: emit SetFee(newBaseFeeX4);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L186

```solidity
File: GeVault.sol

193: emit SetTvlCap(newTvlCap);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L193

```solidity
File: GeVault.sol

240: emit Withdraw(msg.sender, token, amount, liquidity);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L240

```solidity
File: GeVault.sol

283: emit Deposit(msg.sender, token, amount, liquidity);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L283

```solidity
File: GeVault.sol

362: emit Rebalance(tickIndex);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L362

```solidity
File: RangeManager.sol

87: emit AddRange(startX10, endX10, tokenisedRanges.length - 1);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L87

```solidity
File: RangeManager.sol

120: emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);
132: emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L120

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L132



```solidity
File: RangeManager.sol

164: emit Deposit(msg.sender, address(tr), lpAmt);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L164

```solidity
File: RoeRouter.sol

50: emit DeprecatePool(poolId);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RoeRouter.sol#L50

```solidity
File: RoeRouter.sol

78: emit AddPool(poolId, lendingPoolAddressProvider);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RoeRouter.sol#L78

```solidity
File: RoeRouter.sol

86: emit UpdateTreasury(newTreasury);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RoeRouter.sol#L86

```solidity
File: TokenisableRange.sol

119: emit InitTR(address(asset0), address(asset1), startX10, endX10);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L119

```solidity
File: TokenisableRange.sol

162: emit Deposit(msg.sender, 1e18);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L162

```solidity
File: TokenisableRange.sol

214: emit ClaimFees(newFee0, newFee1);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L214

```solidity
File: TokenisableRange.sol

284: emit Deposit(msg.sender, lpAmt);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L284

```solidity
File: TokenisableRange.sol

326: emit Withdraw(msg.sender, lp);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L326

```solidity
File: FixedOracle.sol

26: emit SetHardcodedPrice(_hardcodedPrice);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/FixedOracle.sol#L26

```solidity
File: V3Proxy.sol

121: emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L121

```solidity
File: V3Proxy.sol

134: emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L134

```solidity
File: V3Proxy.sol

143: emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L143

```solidity
File: V3Proxy.sol

157: emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L157

```solidity
File: V3Proxy.sol

175: emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L175

```solidity
File: V3Proxy.sol

193: emit Swap(msg.sender, path[0], path[1], amounts[0], amounts[1]);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L193

```solidity
File: OptionsPositionManager.sol

106: emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1]);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L106

```solidity
File: OptionsPositionManager.sol

148: emit BuyOptions(user, flashAsset, flashAmount, amount0, amount1);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L148

```solidity
File: OptionsPositionManager.sol

240: emit ReducedPosition(user, debtAsset, debt);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L240

```solidity
File: OptionsPositionManager.sol

323: emit ClosePosition(user, debtAsset, debt, amt0, amt1);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L323

```solidity
File: OptionsPositionManager.sol

401: emit SellOptions(msg.sender, optionAddress, deposited, amount0, amount1 );

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L401

```solidity
File: OptionsPositionManager.sol

455: emit Swap(msg.sender, path[0], amount, path[1], received);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L455



</details>




### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Avoid emitting event on every iteration

Expensive operations should always try to be avoided within loops. Such operations include: reading/writing to storage, heavy calculations, external calls, and emitting events. In this instance, an event is being emitted every iteration. Events have a base cost of `Glog (375 gas)` per emit and `Glogdata (8 gas) * number of bytes in event`. We can avoid incurring those costs each iteration by emitting the event outside of the loop.

#### <ins>Proof Of Concept</ins>

```solidity
File: OptionsPositionManager.sol

93: for ( uint8 k =0; k<assets.length; k++){
      address debtAsset = assets[k];
      
      
      uint amount = amounts[k];
      
      
      checkSetAllowance(debtAsset, address(lendingPool), amount);
      lendingPool.liquidationCall(collateral, debtAsset, user, amount, false);
      
      uint debt = closeDebt(poolId, address(this), debtAsset, amount, collateral);
      uint amt0 = ERC20(token0).balanceOf(address(this));
      uint amt1 = ERC20(token1).balanceOf(address(this));
      emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1]);
      amts[0] = amt0;
      amts[1] = amt1;
      
    }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L93



### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

299: for (uint k = 0; k < ticks.length; k++){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L299

```solidity
File: GeVault.sol

314: for (uint k = 0; k < ticks.length; k++){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L314

```solidity
File: GeVault.sol

393: for(uint k=0; k<ticks.length; k++){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L393

```solidity
File: GeVault.sol

431: for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L431

```solidity
File: RangeManager.sol

62: for (uint i = 0; i < len; i++) {

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L62

```solidity
File: OptionsPositionManager.sol

71: for ( uint8 k = 0; k<assets.length; k++){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L71

```solidity
File: OptionsPositionManager.sol

93: for ( uint8 k =0; k<assets.length; k++){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L93

```solidity
File: OptionsPositionManager.sol

172: for (uint8 i = 0; i< options.length; ){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L172

```solidity
File: OptionsPositionManager.sol

203: for (uint8 i = 0; i< options.length; ){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L203



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |



#### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Duplicated `require()`/`revert()` Checks Should Be Refactored To A Modifier Or Function

Saves deployment costs

#### <ins>Proof Of Concept</ins>

```solidity
File: GeVault.sol

120: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
141: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
170: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
203: require(poolMatchesOracle(), "GEV: Oracle Error");
215: require(poolMatchesOracle(), "GEV: Oracle Error");
250: require(poolMatchesOracle(), "GEV: Oracle Error");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L120

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L141

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L170

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L203

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L215

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L250



```solidity
File: V3Proxy.sol

87: require(acceptPayable, "CannotReceiveETH");
91: require(acceptPayable, "CannotReceiveETH");
99: require(path.length == 2, "Direct swap only");
106: require(path.length == 2, "Direct swap only");
113: require(path.length == 2, "Direct swap only");
125: require(path.length == 2, "Direct swap only");
138: require(path.length == 2, "Direct swap only");
148: require(path.length == 2, "Direct swap only");
161: require(path.length == 2, "Direct swap only");
179: require(path.length == 2, "Direct swap only");
139: require(path[0] == ROUTER.WETH9(), "Invalid path");
149: require(path[0] == ROUTER.WETH9(), "Invalid path");
162: require(path[1] == ROUTER.WETH9(), "Invalid path");
180: require(path[1] == ROUTER.WETH9(), "Invalid path");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L87

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L91

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L99

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L106

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L113

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L125

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L138

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L148

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L161

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L179

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L139

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L149

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L162

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L180



```solidity
File: OptionsPositionManager.sol

69: require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");
91: require( address(lendingPool) == msg.sender, "OPM: Call Unallowed");
393: require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );
420: require( LP.getReserveData(optionAddress).aTokenAddress != address(0x0), "OPM: Invalid Address" );

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L69

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L91

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L393

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L420


### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: GeVault.sol

125: require( t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap");
127: require( t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L125

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L127



```solidity
File: GeVault.sol

146: require( t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");
148: require( t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");
158: ticks[0] = t;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L146

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L148

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L158



```solidity
File: RangeManager.sol

111: trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress).balanceOf(msg.sender);
114: LENDING_POOL.getReserveData(address(tokenisedRanges[step])).aTokenAddress,
118: trAmt = LENDING_POOL.withdraw(address(tokenisedRanges[step]), type(uint256).max, address(this));
119: tokenisedRanges[step].withdraw(trAmt, 0, 0);
120: emit Withdraw(msg.sender, address(tokenisedRanges[step]), trAmt);
123: trAmt = ERC20(LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress).balanceOf(msg.sender);
126: LENDING_POOL.getReserveData(address(tokenisedTicker[step])).aTokenAddress,
130: uint256 ttAmt = LENDING_POOL.withdraw(address(tokenisedTicker[step]), type(uint256).max, address(this));
131: tokenisedTicker[step].withdraw(ttAmt, 0, 0);
132: emit Withdraw(msg.sender, address(tokenisedTicker[step]), trAmt);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L111

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L114

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L118

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L119

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L120

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L123

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L126

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L130

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L131

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L132



```solidity
File: V3Proxy.sol

190: weth.withdraw(amounts[1]);
192: payable(msg.sender).call{value: amounts[1]}("");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L190

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L192



```solidity
File: OptionsPositionManager.sol

296: path[0] = token1;
297: path[1] = token0;
301: path[0] = token0;
302: path[1] = token1;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L296

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L297

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L301

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L302





</details>


### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following:

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: GeVault.sol

269: require(tvlCap > valueX8 + getTVL(), "GEV: Max Cap Reached");
220: uint vaultValueX8 = getTVL();
271: uint vaultValueX8 = getTVL();

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L269-L271

```solidity
File: TokenisableRange.sol

225: require(totalSupply() > 0, "TR Closed");
278: lpAmt = totalSupply() * newLiquidity / (liquidity + feeLiquidity);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L225

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L278



```solidity
File: TokenisableRange.sol

294: uint removedLiquidity = uint(liquidity) * lp / totalSupply();
316: TOKEN0.token.safeTransfer( msg.sender, fee0 * lp / totalSupply());
317: removed0 += fee0 * lp / totalSupply();
318: fee0 -= fee0 * lp / totalSupply();
321: TOKEN1.token.safeTransfer(  msg.sender, fee1 * lp / totalSupply());
322: removed1 += fee1 * lp / totalSupply();
323: fee1 -= fee1 * lp / totalSupply();

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L294

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L316

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L317

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L318

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L321

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L322

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L323



```solidity
File: TokenisableRange.sol

353: if ( totalSupply() == 0 ) return 0;
356: return totalValue * 1e18 / totalSupply();

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L353

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L356



```solidity
File: TokenisableRange.sol

379: token0Amount += fee0 * amount / totalSupply();
380: token1Amount += fee1 * amount / totalSupply();

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L379-L380

```solidity
File: V3Proxy.sol

162: require(path[1] == ROUTER.WETH9(), "Invalid path");
180: require(path[1] == ROUTER.WETH9(), "Invalid path");
170: IWETH9 weth = IWETH9(ROUTER.WETH9());
188: IWETH9 weth = IWETH9(ROUTER.WETH9());

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L162

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L180

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L170

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L188

</details>


### <a href="#gas-summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Splitting `require()` statements that use `&&` saves gas

Instead of using operator `&&` on a single `require`. Using a two `require` can save more gas.

i.e.
for `require(version == 1 && _bytecodeHash[1] == bytes1(0), "zf");` use:

```
	require(version == 1);
	require(_bytecodeHash[1] == bytes1(0));
```

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

120: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L120

```solidity
File: GeVault.sol

141: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L141

```solidity
File: GeVault.sol

170: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L170

```solidity
File: RangeManager.sol

108: require(step < tokenisedRanges.length && step < tokenisedTicker.length, "Invalid step");
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L108

```solidity
File: TokenisableRange.sol

209: require(addedValue > liquidityValue * 95 / 100 && liquidityValue > addedValue * 95 / 100, "TR: Claim Fee Slippage");
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L209

```solidity
File: LPOracle.sol

29: require(lpToken != address(0x0) && clToken0 != address(0x0) && clToken1 != address(0x0), "Invalid address");
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/LPOracle.sol#L29

```solidity
File: LPOracle.sol

101: require(decimalsA <= 18 && decimalsB <= 18, "Incorrect tokens");
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/LPOracle.sol#L101

```solidity
File: OptionsPositionManager.sol

167: require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L167

```solidity
File: OptionsPositionManager.sol

495: require( amountsIn[0] <= maxAmount && amountsIn[0] > 0, "OPM: Invalid Swap Amounts" );
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L495

```solidity
File: OptionsPositionManager.sol

536: require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");
```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L536



</details>



### <a href="#gas-summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

#### <ins>Proof Of Concept</ins>

```solidity
File: V3Proxy.sol

22: struct ExactOutputSingleParams {
        address tokenIn;
        address tokenOut;
        uint24 fee;
        address recipient;
        --- uint256 deadline;
        +++ uint64 deadline; // @audit can be packed to lower uint64 as it is unlikely for deadline to ever reach uint64.max
        uint256 amountOut;
        uint256 amountInMaximum;
        uint160 sqrtPriceLimitX96;
    }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L22



### <a href="#gas-summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

298: function getReserves

for (uint k = 0; k < ticks.length; k++){
      TokenisableRange t = ticks[k];
      address aTick = lendingPool.getReserveData(address(t)).aTokenAddress;
      uint bal = ERC20(aTick).balanceOf(address(this));
      (uint amt0, uint amt1) = t.getTokenAmounts(bal);
      amount0 += amt0;
      amount1 += amt1;
    }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L298

```solidity
File: GeVault.sol

313: function removeFromAllTicks

for (uint k = 0; k < ticks.length; k++){
      removeFromTick(k);
    }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L313

```solidity
File: GeVault.sol

392: function getTVL

for(uint k=0; k<ticks.length; k++){
      TokenisableRange t = ticks[k];
      uint bal = getTickBalance(k);
      valueX8 += bal * t.latestAnswer() / 1e18;
    }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L392

```solidity
File: GeVault.sol

428: function getActiveTickIndex

for (activeTickIndex = 0; activeTickIndex < ticks.length - 3; activeTickIndex++){
        (uint amt0, uint amt1) = ticks[activeTickIndex+1].getTokenAmountsExcludingFees(1e18);
        (uint amt0n, uint amt1n) = ticks[activeTickIndex+2].getTokenAmountsExcludingFees(1e18);
        if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )
          break;
      }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L428

```solidity
File: RangeManager.sol

59: function checkNewRange

for (uint i = 0; i < len; i++) {
      if (start >= stepList[i].end || end <= stepList[i].start) {
        continue;
      }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L59

```solidity
File: OptionsPositionManager.sol

61: function executeBuyOptions

for ( uint8 k = 0; k<assets.length; k++){
      address asset = assets[k];
      uint amount = amounts[k];
      withdrawOptionAssets(poolId, asset, amount, sourceSwap[k], user);
    }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L62

```solidity
File: OptionsPositionManager.sol

83: function executeLiquidation

for ( uint8 k =0; k<assets.length; k++){
      address debtAsset = assets[k];
      
      
      uint amount = amounts[k];
      
      
      checkSetAllowance(debtAsset, address(lendingPool), amount);
      lendingPool.liquidationCall(collateral, debtAsset, user, amount, false);
      
      uint debt = closeDebt(poolId, address(this), debtAsset, amount, collateral);
      uint amt0 = ERC20(token0).balanceOf(address(this));
      uint amt1 = ERC20(token1).balanceOf(address(this));
      emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1]);
      amts[0] = amt0;
      amts[1] = amt1;
      
    }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L84

```solidity
File: OptionsPositionManager.sol

159: function buyOptions

for (uint8 i = 0; i< options.length; ){
      flashtype[i] = 2;
      unchecked { i+=1; }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L160

```solidity
File: OptionsPositionManager.sol

189: function liquidate 

for (uint8 i = 0; i< options.length; ){
      flashtype[i] = 0; 
      unchecked { i+=1; }

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L190



</details>


### <a href="#gas-summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Use nested `if` and avoid multiple check combinations

Using nested `if`, is cheaper than using `&&` multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

377: if (oraclePrice < priceX8 * 101 / 100 && oraclePrice > priceX8 * 99 / 100) matches = true;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L377

```solidity
File: GeVault.sol

434: if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L434

```solidity
File: TokenisableRange.sol

177: if ((newFee0 == 0) && (newFee1 == 0)) return;
190: if ((fee0 * 100 > bal0 ) && (fee1 * 100 > bal1)) {

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L177

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L190



```solidity
File: TokenisableRange.sol

237: if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) ){
268: if ( newFee0 == 0 && newFee1 == 0 ){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L237

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L268



```solidity
File: OptionsPositionManager.sol

234: if ( repayAmount > 0 && repayAmount < debt ) debt = repayAmount;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L234

```solidity
File: OptionsPositionManager.sol

473: if (amount > 0 && AmountsRouter(address(ammRouter)).getAmountsOut(amount, path)[1] > 0){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L473



</details>


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
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }
    
    }
    
    contract Contract1 {
    
        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |



### <a href="#gas-summary">[GAS&#x2011;14]</a><a name="GAS&#x2011;14"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: GeVault.sol

120: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
121: if (ticks.length == 0) ticks.push(t);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L120-L121

```solidity
File: GeVault.sol

141: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
142: if (ticks.length == 0) ticks.push(t);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L141-L142

```solidity
File: GeVault.sol

170: require(t0 == token0 && t1 == token1, "GEV: Invalid TR");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L170

```solidity
File: GeVault.sol

216: if (liquidity == 0) liquidity = balanceOf(msg.sender);
223: uint fee = amount * getAdjustedBaseFee(token == address(token1)) / 1e4;
230: if (token == address(WETH)){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L216

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L223

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L230



```solidity
File: GeVault.sol

251: require(token == address(token0) || token == address(token1), "GEV: Invalid Token");
252: require(amount > 0 || msg.value > 0, "GEV: Deposit Zero");
256: require(token == address(WETH), "GEV: Invalid Weth");
266: uint fee = amount * getAdjustedBaseFee(token == address(token0)) / 1e4;
274: if (tSupply == 0 || vaultValueX8 == 0)

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L251

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L252

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L256

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L266

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L274



```solidity
File: GeVault.sol

291: if (supply == 0) return 0;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L291

```solidity
File: GeVault.sol

434: if ( (amt0 == 0 && amt0n > 0) || (amt1 == 0 && amt1n > 0) )

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/GeVault.sol#L434

```solidity
File: RangeManager.sol

63: if (start >= stepList[i].end || end <= stepList[i].start) {

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/RangeManager.sol#L63

```solidity
File: TokenisableRange.sol

87: require(status == ProxyState.INIT_PROXY, "!InitProxy");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L87

```solidity
File: TokenisableRange.sol

135: require(status == ProxyState.INIT_LP, "!InitLP");
136: require(msg.sender == creator, "Unallowed call");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L135-L136

```solidity
File: TokenisableRange.sol

177: if ((newFee0 == 0) && (newFee1 == 0)) return;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L177

```solidity
File: TokenisableRange.sol

237: if ( fee0+fee1 > 0 && ( n0 > 0 || fee0 == 0) && ( n1 > 0 || fee1 == 0 ) ){
268: if ( newFee0 == 0 && newFee1 == 0 ){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L237

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L268



```solidity
File: TokenisableRange.sol

335: if (TOKEN0_PRICE == 0) TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token));
336: if (TOKEN1_PRICE == 0) TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token));

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/TokenisableRange.sol#L335-L336

```solidity
File: V3Proxy.sol

99: require(path.length == 2, "Direct swap only");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L99

```solidity
File: V3Proxy.sol

106: require(path.length == 2, "Direct swap only");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L106

```solidity
File: V3Proxy.sol

113: require(path.length == 2, "Direct swap only");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L113

```solidity
File: V3Proxy.sol

125: require(path.length == 2, "Direct swap only");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L125

```solidity
File: V3Proxy.sol

138: require(path.length == 2, "Direct swap only");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L138

```solidity
File: V3Proxy.sol

148: require(path.length == 2, "Direct swap only");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L148

```solidity
File: V3Proxy.sol

161: require(path.length == 2, "Direct swap only");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L161

```solidity
File: V3Proxy.sol

179: require(path.length == 2, "Direct swap only");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/helper/V3Proxy.sol#L179

```solidity
File: OptionsPositionManager.sol

47: if ( mode == 0 ){

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L47

```solidity
File: OptionsPositionManager.sol

137: require(sourceSwap == token0 || sourceSwap == token1, "OPM: Invalid Swap Token");
140: path[1] = sourceSwap == token0 ? token1 : token0;
141: uint amount = sourceSwap == token0 ? amount0 : amount1;
145: amount0 = sourceSwap == token0 ? 0 : amount0 + received;
146: amount1 = sourceSwap == token1 ? 0 : amount1 + received;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L137

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L140

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L141

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L145

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L146



```solidity
File: OptionsPositionManager.sol

167: require(options.length == amounts.length && sourceSwap.length == options.length, "OPM: Array Length Mismatch");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L167

```solidity
File: OptionsPositionManager.sol

198: require(options.length == amounts.length, "ARRAY_LEN_MISMATCH");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L198

```solidity
File: OptionsPositionManager.sol

261: require(collateralAsset == token0 || collateralAsset == token1, "OPM: Invalid Collateral Asset");
283: if (collateralAsset == token0) amtA -= feeAmount;
327: if (user == msg.sender) swapTokens(poolId, collateralAsset == token0 ? token1 : token0, 0);

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L261

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L283

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L327



```solidity
File: OptionsPositionManager.sol

441: require(sourceAsset == token0 || sourceAsset == token1, "OPM: Invalid Swap Asset");
442: if (amount == 0) {
444: if (amount == 0) return 0;
450: path[1] = sourceAsset == token0 ? token1 : token0;

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L441

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L442

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L444

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L450



```solidity
File: OptionsPositionManager.sol

536: require(token0 == address(t0) && token1 == address(t1), "OPM: Invalid Debt Asset");

```

https://github.com/code-423n4/2023-08-goodentry/tree/main/contracts/PositionManager/OptionsPositionManager.sol#L536



</details>


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

#### Gas Test Report

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
