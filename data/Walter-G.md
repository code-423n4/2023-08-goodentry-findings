### OptionsPositionManager.sol
we could do it using assembly :D
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L203-L206
assembly (new version of code):
```
{
    let length := sload(options_slot)
    let i := 0

    for { } lt(i, length) { } {
        // Set the value at the flashtype[i] storage slot to 0
        sstore(add(options_slot, i), 0)
        i := add(i, 1)
    }
}
```
2) 233-234 we could re-write them using yul
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L233-L234
```
{
    let variableDebtTokenAddress := sload(add(debtAsset_slot, 1))
    let debt := sload(variableDebtTokenAddress_slot)
    let repayAmount := sload(repayAmount_slot)
    if and(gt(repayAmount, 0), lt(repayAmount, debt)) {
        debt := repayAmount
    }
}
```
3) all of this could be re-write it in yul as usual :D 
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L312-L323
```
amt0 := sload(token0_slot)
amt1 := sload(token1_slot)

// Check for edge case where dust causes remaining asset balance to be slightly higher than before repaying
if gt(amtA, amt0) {
    amt0 := sub(amtA, amt0)
} else {
    amt0 := 0
}

if gt(amtB, amt1) {
    amt1 := sub(amtB, amt1)
} else {
    amt1 := 0
}

// Emit the ClosePosition event
datacopy(0, user, 0x20)
datacopy(0x20, debtAsset, 0x20)
datacopy(0x40, debt, 0x20)
datacopy(0x60, amt0, 0x20)
datacopy(0x80, amt1, 0x20)
mstore(0xa0, 0x20)  // Length of the data
mstore(0xc0, 0x08)  // ClosePosition function signature
log2(0xa0, 0x100)   // Emit the ClosePosition event
```
4) 546-550 yul??
https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/PositionManager/OptionsPositionManager.sol#L546-L550
```
let token0Decimals := sload(token0_decimals_slot)
let token1Decimals := sload(token1_decimals_slot)
let token0Price := sload(token0_price_slot)
let token1Price := sload(token1_price_slot)

let scale0 := div(mul(exp(2, sub(20, token0Decimals)), token0Price), 0x64)
let scale1 := div(mul(exp(2, sub(20, token1Decimals)), token1Price), 0x64)

// Compare scale0 and scale1
switch lt(scale0, scale1)
case 1 {
    amount := scale0
}
default {
    amount := scale1
}
```