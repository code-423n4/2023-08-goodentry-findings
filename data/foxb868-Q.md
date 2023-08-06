## While the usage of `address(0x0)` for input validation is a common practice, it is advisable to prioritize code readability and clarity by employing the `address(0)` notation instead.

## Impact

Using `address(0x0)` to validate input addresses may introduce confusion and make the code less readable, particularly for developers who are not familiar with the hexadecimal representation of the zero address. This can result in errors, difficulty in understanding the codebase, and even unintended vulnerabilities if developers misinterpret the usage.

The vulnerability related to using `address(0x0)` to check for the validity of inputs is present in the following lines of code in the `RoeRouter` contract [#Line34](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/RoeRouter.sol#L34)

```solidity
require(treasury_ != address(0x0), "Invalid address");
```

```solidity
require (
  lendingPoolAddressProvider != address(0x0) 
  && token0 != address(0x0) 
  && token1 != address(0x0) 
  && ammRouter != address(0x0), 
  "Invalid Address"
);
```

## Proof of Concept

Using `address(0x0)` for Validation.
```solidity
pragma solidity ^0.8.0;

contract AddressValidationExample {
    address public validAddress;

    function setAddress(address _addr) public {
        require(_addr != address(0x0), "Invalid address");
        validAddress = _addr;
    }
}
```

Using `address(0)` for Validation (Recommended):
```solidity
pragma solidity ^0.8.0;

contract AddressValidationExample {
    address public validAddress;

    function setAddress(address _addr) public {
        require(_addr != address(0), "Invalid address");
        validAddress = _addr;
    }
}
```
Now to explain Explanation.
In both examples, we have a smart contract `AddressValidationExample` with a function `setAddress` that accepts an Ethereum address `_addr` as input and sets it as `validAddress` if it is considered valid.

In the first example, we use `address(0x0)` to represent the zero address and check if the input address is not equal to the zero address to validate it. This is a common practice, but it might not be as clear for developers who are not familiar with the hexadecimal representation of the zero address.

In the second example, we use `address(0)` to represent the zero address, which is arguably more readable and clear, especially for developers who may not be well-versed in hexadecimal notation.

Using `address(0)` is recommended for better clarity and readability, as it directly represents the zero address and avoids potential confusion.

Both examples achieve the same result in terms of validation. However, the second example using `address(0)` is easier to understand, making the code more approachable and reducing the risk of misunderstandings.

## Recommended Mitigation Steps

Recommended practice for improved clarity and readability is to use the `address(0)` notation instead. Here is how the lines of code should be updated:

```solidity
require(treasury_ != address(0), "Invalid address");
```

```solidity
require (
  lendingPoolAddressProvider != address(0) 
  && token0 != address(0) 
  && token1 != address(0) 
  && ammRouter != address(0), 
  "Invalid Address"
);
```
By making this change, the code becomes more explicit and easier to understand, which can help prevent potential misunderstandings related to the use of `address(0x0)`.

** **

## The redundant initial check may result in slightly higher gas consumption during contract execution due to unnecessary computations.

## Impact
The redundant initial check does not cause any critical issues or security risks in the smart contract. It may result in slightly higher gas consumption due to unnecessary computations and code execution, but it does not affect the overall functionality of the contract.

In both `pushTick` and `shiftTick` functions, there is an initial check for adding the first `tokenizable` range to the `ticks` array. However, the same check is performed again after that. The initial check seems redundant and can be removed.

## Proof of Concept

**In the [pushTick](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L116-L132) function**
```solidity
function pushTick(address tr) public onlyOwner {
    TokenisableRange t = TokenisableRange(tr);
    (ERC20 t0,) = t.TOKEN0();
    (ERC20 t1,) = t.TOKEN1();
    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
    // Redundant initial check for adding the first tokenizable range
    if (ticks.length == 0) ticks.push(t);
    else {
        // Check that tick is properly ordered
        if (baseTokenIsToken0) 
            require( t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap");
        else 
            require( t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap");
        ticks.push(TokenisableRange(tr));
    }
    emit PushTick(tr);
}
```

**In the [shiftTick](https://github.com/code-423n4/2023-08-goodentry/blob/71c0c0eca8af957202ccdbf5ce2f2a514ffe2e24/contracts/GeVault.sol#L137-L161) function**
```solidity
function shiftTick(address tr) public onlyOwner {
    TokenisableRange t = TokenisableRange(tr);
    (ERC20 t0,) = t.TOKEN0();
    (ERC20 t1,) = t.TOKEN1();
    require(t0 == token0 && t1 == token1, "GEV: Invalid TR");
    // Redundant initial check for adding the first tokenizable range
    if (ticks.length == 0) ticks.push(t);
    else {
        // Check that tick is properly ordered
        if (!baseTokenIsToken0) 
            require( t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");
        else 
            require( t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");
        // extend array by pushing last elt
        ticks.push(ticks[ticks.length-1]);
        // shift each element
        if (ticks.length > 2){
            for (uint k = 0; k < ticks.length - 2; k++) 
                ticks[ticks.length - 2 - k] = ticks[ticks.length - 3 - k];
        }
        // add new tick in first place
        ticks[0] = t;
    }
    emit ShiftTick(tr);
}
```

The initial check for the `ticks` array's length is performed at the beginning of both functions using the condition `if (ticks.length == 0)`. However, the same check is repeated later in the function body with additional logic for adding the tokenizable range to the `ticks` array. The initial check is redundant and can be safely removed without affecting the intended functionality of the functions.

## Recommended Mitigation Steps
To improve code efficiency and optimize gas consumption, the redundant initial check in the `pushTick` and `shiftTick` functions can be safely removed.