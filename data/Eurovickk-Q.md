https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

1)Tick Management and Range Handling:
Issue: Overlapping Ticks in pushTick and shiftTick
function pushTick(address tr) public onlyOwner {
    // ...
    if (baseTokenIsToken0) 
        require(t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap");
    else 
        require(t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap");
    // ...
}
function shiftTick(address tr) public onlyOwner {
    // ...
    if (!baseTokenIsToken0) 
        require(t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");
    else 
        require(t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");
    // ...
}
The pushTick and shiftTick functions check for tick overlap with existing ticks. However, the condition to check for overlap may not be sufficient to prevent all possible cases of overlap, especially when considering rounding or edge cases.

Check for Lower and Upper Bounds of Overlap:
When checking for tick overlap, consider the lower and upper bounds of the tick ranges involved. Ensure that the new tick's lower bound is strictly greater than the existing tick's upper bound, and the new tick's upper bound is strictly less than the existing tick's lower bound. This approach helps prevent rounding and boundary issues from causing overlap.
// For pushTick
if (baseTokenIsToken0) {
    require(t.lowerTick() > ticks[ticks.length-1].upperTick(), "GEV: Push Tick Overlap");
} else {
    require(t.upperTick() < ticks[ticks.length-1].lowerTick(), "GEV: Push Tick Overlap");
}
// For shiftTick
if (!baseTokenIsToken0) {
    require(t.lowerTick() > ticks[0].upperTick(), "GEV: Shift Tick Overlap");
} else {
    require(t.upperTick() < ticks[0].lowerTick(), "GEV: Shift Tick Overlap");
}
Consider Rounding and Precision:
Since Solidity's arithmetic involves rounding, be cautious when comparing tick values that are close together. You might need to adjust the comparison conditions slightly to account for potential rounding errors.
// Adjusted check for pushTick
if (baseTokenIsToken0) {
    require(t.lowerTick() > ticks[ticks.length-1].upperTick() + 1, "GEV: Push Tick Overlap");
} else {
    require(t.upperTick() < ticks[ticks.length-1].lowerTick() - 1, "GEV: Push Tick Overlap");
}
// Adjusted check for shiftTick
if (!baseTokenIsToken0) {
    require(t.lowerTick() > ticks[0].upperTick() + 1, "GEV: Shift Tick Overlap");
} else {
    require(t.upperTick() < ticks[0].lowerTick() - 1, "GEV: Shift Tick Overlap");
}
Test with Boundary Values:
When testing the tick overlap conditions, include test cases with boundary values to ensure that the contract behaves correctly when ticks are very close or just outside the boundaries.


2)Token Approval:
Issue: Missing Allowance Revocation in depositAndStash
function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity) {
    // ...
    liquidity = t.deposit(amount0, amount1);

    uint bal = t.balanceOf(address(this));
    if (bal > 0) {
        checkSetApprove(address(t), address(lendingPool), bal);
        lendingPool.deposit(address(t), bal, address(this), 0);
    }
    // ...
}
The function depositAndStash interacts with external contracts and sets allowances using the checkSetApprove function. However, it doesn't revoke the allowance after the interaction is complete.  An attacker could take advantage of the fact that the allowance is not revoked after the interaction, potentially allowing unauthorized access to the contract's tokens if the contract's state changes in certain ways.
function depositAndStash(TokenisableRange t, uint amount0, uint amount1) internal returns (uint liquidity) {
    // ... (previous code)

    uint bal = t.balanceOf(address(this));
    if (bal > 0) {
        checkSetApprove(address(t), address(lendingPool), bal);
        lendingPool.deposit(address(t), bal, address(this), 0);
        
        // Revoke the allowance after interaction
        ERC20(t).approve(address(lendingPool), 0);
    }
    // ... (remaining code)
}
By adding the line ERC20(t).approve(address(lendingPool), 0);, you revoke the allowance immediately after the interaction with the external contract is complete.

3)Fallback Function (receive()):
Issue: Fallback Function Interaction with Other Parts of the Contract
receive() external payable {
    if(msg.sender != address(WETH)) deposit(address(WETH), msg.value);
}
The fallback function conditionally calls the deposit function based on the sender's address. Depending on how deposit and other contract functions are implemented, unintended interactions or reentrancy vulnerabilities could arise.

https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol

1)Oracle Price Validation: Validate that the retrieved oracle prices (TOKEN0_PRICE and TOKEN1_PRICE) are non-zero before using them in calculations to avoid division by zero and other potential issues.

function calculateValue(uint amount) external view returns (uint256 value) {
    uint256 TOKEN0_PRICE = ORACLE.getAssetPrice(address(TOKEN0.token));
    uint256 TOKEN1_PRICE = ORACLE.getAssetPrice(address(TOKEN1.token));
    
    require(TOKEN0_PRICE > 0, "Invalid TOKEN0_PRICE");
    require(TOKEN1_PRICE > 0, "Invalid TOKEN1_PRICE");
    
    // Perform calculations using non-zero oracle prices
    // Example calculation: value = amount * (TOKEN0_PRICE / TOKEN1_PRICE)
    value = amount * TOKEN0_PRICE / TOKEN1_PRICE;
}
In this proof of concept, we first retrieve the oracle prices for TOKEN0 and TOKEN1 using the ORACLE.getAssetPrice function. Then, we use require statements to ensure that both TOKEN0_PRICE and TOKEN1_PRICE are greater than zero before performing any calculations involving division. This helps prevent division by zero errors.

2)Input Validation: Many external inputs, such as n0, n1, lp, amount0Min, and amount1Min, are used in calculations without proper validation. You should validate that these inputs are within acceptable ranges and handle potential edge cases to prevent unexpected behavior.
In the deposit function, you are transferring tokens from the sender to the contract and then using these values for further calculations. It's important to ensure that the transferred token amounts are valid and within acceptable ranges. Here's how you can validate the n0 and n1 inputs:
function deposit(uint256 n0, uint256 n1) external nonReentrant returns (uint256 lpAmt) {
    require(totalSupply() > 0, "TR Closed");
    claimFee();

    // Validate input values
    require(n0 > 0, "Invalid n0");
    require(n1 > 0, "Invalid n1");

    TOKEN0.token.transferFrom(msg.sender, address(this), n0);
    TOKEN1.token.transferFrom(msg.sender, address(this), n1);

    // ... rest of the function ...
}
Validation of Inputs: In the code snippet above, we added input validation for n0 and n1 to ensure that they are both greater than zero. This prevents the function from proceeding if either of these input values is not within an acceptable range.
Preventing Invalid Inputs: Without proper input validation, malicious actors or unintentional errors could provide invalid or zero token amounts, which might lead to unexpected behavior or even exploits in your contract.
Ensuring Expected Behavior: By validating inputs, you ensure that the function only proceeds with valid and reasonable inputs, reducing the risk of incorrect calculations or unwanted transfers.
Preventing Reentrancy Attacks: Adding input validation at the beginning of the function helps prevent reentrancy attacks, as it ensures that the function cannot be called again with invalid inputs during its execution.
By implementing input validation, you create a more robust and secure smart contract that can handle various scenarios and prevent unexpected behavior caused by invalid or out-of-range inputs.

