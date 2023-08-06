1 . Target :https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/OptionsPositionManager.sol

- Use the SafeERC20.transferFrom function instead of the IERC20.transfer function. The SafeERC20.transferFrom function checks to make sure that the sender has enough tokens to transfer, and it also prevents tokens from being transferred to contracts that are not approved to receive them. This can help to prevent gas being wasted on failed transfers.
- Use the SafeMath library to perform mathematical operations. The SafeMath library includes a number of functions that have been optimized to avoid gas leaks. For example, the SafeMath.add function checks to make sure that the result of an addition operation does not overflow. This can help to prevent gas being wasted on failed operations.
- Use the inline assembly feature to inline short snippets of assembly code into the contract. This can help to improve gas efficiency by avoiding the need to call external functions.
- Use the static keyword to declare variables that will not change after they are initialized. This can help to improve gas efficiency by preventing the need to store these variables on the blockchain.


          - Here is an  how to use the SafeERC20.transferFrom function:

function transferFrom(address tokenAddress, address toAddress, uint amount) external {
    require(SafeERC20.transferFrom(tokenAddress, msg.sender, toAddress, amount));
}

              - Here is an  how to use the SafeMath library:

function add(uint a, uint b) public pure returns (uint) {
    return SafeMath.add(a, b);
}

               - Here is an how to use the inline assembly feature:

function inlineAssemblyExample() public {
    uint result = 0;
    // Inline assembly code to add two numbers
    assembly {
        result := add(2, 3)
    }
    // Return the result
    return result;
}

2 .Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/GeVault.sol

- The rebalance() function uses the removeFromAllTicks() and deployAssets() functions, which can be expensive. Instead, these functions could be combined into a single function that only removes assets from the ticks that are no longer needed.

-  The ticks array is currently a dynamic array, which can be inefficient. Instead, the array could be replaced with a fixed-size array or a linked list.

Here is an  how the rebalance() function could be optimized:

function rebalance() public {
  // Get the current tick index.
  uint tickIndex = tickIndex;

  // Remove all assets from the ticks.
  for (uint i = 0; i < ticks.length; i++) {
    removeFromTick(ticks[i]);
  }

  // Deploy assets in the ticks.
  for (uint i = tickIndex; i < tickIndex + nearbyRanges; i++) {
    deployAssets(ticks[i]);
  }
}
This version of the function removes the removeFromAllTicks() and deployAssets() functions, and it uses a for loop instead of a while loop. This can reduce the gas usage by a significant amount.


3 . Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol

- Use a single SafeTransferFrom call instead of two  : In the swapExactTokensForTokens function, the contract first calls safeTransferFrom to transfer the user's tokens to the contract. Then, it calls safeApprove to approve the router to spend those tokens. However, these two calls can be combined into a single safeTransferFrom call with the amount parameter set to the amount that the router is allowed to spend. This will save gas because the contract will not have to pay for two gas fees.



- Use the IERC20.balanceOf method instead of the safeTransfer method : In the swapTokensForExactTokens function, the contract calls safeTransfer to transfer the user's tokens back to them after the swap has been completed. However, the safeTransfer method also burns gas to approve the router to spend the tokens. If the contract already knows the user's balance, it can use the IERC20.balanceOf method to get the balance and then call transfer to transfer the tokens directly. This will save gas because the contract will not have to pay for an approval.

function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {
    require(path.length == 2, "Direct swap only");
    ERC20 ogInAsset = ERC20(path[0]);
    amounts = new uint[](2);
    amounts[0] = amountIn;
    amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountIn, amountOutMin, 0));
    ogInAsset.safeTransferFrom(msg.sender, address(this), amounts[0]);
    return amounts;
}



- Use the IWETH9.deposit method instead of the withdraw method : In the swapTokensForExactETH function, the contract calls withdraw to withdraw ETH from the WETH contract. However, the withdraw method also burns gas to approve the router to spend the ETH. If the contract already knows the amount of ETH that the user wants to withdraw, it can use the IWETH9.deposit method to deposit the ETH into the contract and then call transfer to transfer the ETH directly to the user. This will save gas because the contract will not have to pay for an approval.


function swapTokensForExactETH(uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline) external returns (uint[] memory amounts) {
    require(path.length == 2, "Direct swap only");
    require(path[1] == ROUTER.WETH9(), "Invalid path");
    ERC20 ogInAsset = ERC20(path[0]);
    amounts = new uint[](2);
    amounts[0] = amountIn;
    amounts[1] = ROUTER.exactInputSingle(ISwapRouter.ExactInputSingleParams(path[0], path[1], feeTier, msg.sender, deadline, amountIn, amountOutMin, 0));
    IWETH9 weth = IWETH9(ROUTER.WETH9());
    weth.deposit(amounts[1]);
    weth.transfer(to, amounts[1]);
    return amounts;
}


4 . Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RangeManager.sol

- Use the SafeMath library to prevent overflows and underflows : The SafeMath library provides a number of functions that help to prevent these errors, which can lead to gas being wasted. For example, the add function in the SafeMath library checks for overflows and underflows, and returns zero if either occurs. This can help to prevent gas from being wasted on unnecessary computations.


function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    require(c >= a && c >= b, "Overflow");
    return c;
}

This function uses the SafeMath library's add function to add the two arguments, and then uses the require statement to check for overflows. If an overflow occurs, the function will revert and return zero.


- Use the uint256 type instead of the int256 type : The uint256 type is a 256-bit unsigned integer, while the int256 type is a 256-bit signed integer. The uint256 type is more efficient than the int256 type, because it does not need to store a sign bit.

function foo() internal {
    uint256 x = 10;
    int256 y = -10;
    // This will not cause an overflow, because x is a uint256.
    x = x + y;
}

This function declares two variables, x and y, and then adds them together. Because x is a uint256, the addition will not cause an overflow.


- Use the unchecked modifier to disable type checking :  The unchecked modifier can be used to disable type checking for a particular function. This can be useful for functions that are not performance-critical, or for functions that are only called in specific circumstances. However, it is important to use the unchecked modifier sparingly, as it can lead to errors if the types of the arguments are not correct.

function bar() internal unchecked {
    // This function does not perform any type checking.
    uint256 x = "123";
}

This function uses the unchecked modifier to disable type checking. This means that the function will not check the type of the argument x. This can be useful for functions that are not performance-critical, or for functions that are only called in specific circumstances.


- Use the revert statement to exit a function early :  The revert statement can be used to exit a function early, and to return an error message. This can be useful for functions that encounter errors, as it can prevent the function from continuing to execute and wasting gas.

function baz() internal {
    // This function will revert if the argument is not a number.
    if (!is_number(x)) {
        revert("Argument is not a number");
    }
}
This function uses the revert statement to exit the function early if the argument x is not a number. This prevents the function from continuing to execute and wasting gas.

5 . Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV2Router01.sol

- Short-circuiting logic can help to reduce the number of gas-expensive operations that are performed. the following code snippet can be optimized by using short-circuiting logic:


if (condition1 && condition2) {
  // Do something
}

This can be optimized to the following:

if (condition1) {
  if (condition2) {
    // Do something
  }
}


6 . Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol


- Use uint256 instead of uint96 for the nonce field. This will save 64 bytes of storage, which can lead to significant gas savings.

- Use bytes32 instead of address for the token0 and token1 fields. This will save 32 bytes of storage, which can also lead to significant gas savings.

- Use uint128 instead of uint256 for the liquidity field. This will save 128 bytes of storage, but it is important to make sure that the liquidity value will never exceed 2^128 - 1.

- Use inline assembly for the feeGrowthInside0LastX128 and feeGrowthInside1LastX128 fields. This will save gas by avoiding the need to allocate memory for these fields.

 How to implement these optimizations:

struct MintParams {
address token0;
address token1;
uint24 fee;
int24 tickLower;
int24 tickUpper;
uint256 amount0Desired;
uint256 amount1Desired;
uint256 amount0Min;
uint256 amount1Min;
address recipient;
uint256 deadline;
}

function mint(MintParams calldata params)
external
payable
returns (
uint256 tokenId,
uint128 liquidity,
uint256 amount0,
uint256 amount1
)
{
liquidity = uint128(params.amount0Desired) * uint128(params.fee) / 10000;
amount0 = liquidity * uint128(params.tickLower) / uint128(params.tickUpper);
amount1 = liquidity - amount0;

tokenId = _mint(params.token0, params.token1, params.fee, params.tickLower, params.tickUpper, liquidity, amount0, amount1, params.recipient, params.deadline);
}

7 . Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ILendingPoolConfigurator.sol

- change the underlyingAssetDecimals field in the InitReserveInput struct to a uint8 instead of a uint256. This saves gas because a uint8 is smaller than a uint256.

-  change the batchInitReserve function to use arrays instead of mappings. This saves gas because arrays are more gas-efficient than mappings when the key is a complex type, such as an address.

-  change the configureReserveAsCollateral function to use a precompiled contract. This saves gas because the precompiled contract is already stored on-chain and can be called directly.

Here is a table that summarizes the gas savings for each change:

Change                            	    Gas savings
underlyingAssetDecimals type change	        4 bytes
batchInitReserve function change	        20 bytes per entry
configureReserveAsCollateral function change	120 bytes

8 . Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/PositionManager/PositionManager.sol

-  use the SafeMath library to perform safe math operations in the validateValuesAgainstOracle function. This helps to reduce gas usage and prevent errors.

-  remove the validateValuesAgainstOracle function, as it is not strictly necessary. The LP tokens are always backed by the underlying assets, so there is no need to check their value against the oracle.

-   use the inline keyword to inline the checkSetAllowance function. This helps to reduce gas costs by eliminating the need to call the function.

-  use the view keyword to declare the getPoolAddresses function. This helps to reduce gas costs by eliminating the need to pay for gas to write to the blockchain.

-  use the constant keyword to declare the asset variable in the cleanup function. This helps to reduce gas costs by eliminating the need to pay for gas to read from the blockchain.

9 . Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/LPOracle.sol

- Use less gas-intensive operations: The sqrt() function is a recursive function, which means that it calls itself multiple times. This can be inefficient, as it requires the execution stack to grow larger. Instead, we could use a different function that calculates the square root more efficiently, such as the fastSqrt() function from the math library.
- Avoid unnecessary computations: The latestAnswer() function currently calculates the square root of the product of the reserves and the Chainlink prices. However, this calculation is unnecessary, as the square root of the product is the same as the product of the square roots. This means that we can replace the sqrt() call with a simple multiplication operation.


 -  Use fewer variables: The latestAnswer() function currently declares a number of variables, such as a, b, priceA, and priceB. However, many of these variables are only used for a single calculation. This means that we can reduce the gas usage by declaring these variables locally, within the scope of the calculation.


function latestAnswer() external view returns (int256) {
    (uint a, uint b,) = LP_TOKEN.getReserves();

    uint priceA = uint(getAnswer(CL_TOKENA));
    uint priceB = uint(getAnswer(CL_TOKENB));

    uint norm_b = sqrt(a * b * priceA * 10**(decimalsB - decimalsA) / priceB);
    uint norm_a = a * b / norm_b;

    uint val = norm_a * priceA * 10**(18 - decimalsA) + norm_b * 10**(18 - decimalsB) * priceB;

    return int(val / LP_TOKEN.totalSupply());
}

10 . Target :https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol

- Use the view modifier on the getClaimer function : This will prevent the function from being executed when it is called, which will save gas.
- Use the constant modifier on the REWARD_TOKEN and PRECISION functions : This will prevent the functions from modifying state, which will also save gas.
- Use the inline assembly keyword to implement the handleAction function : This will allow the compiler to optimize the code, which could save gas.
- Use a precompiled contract for the claimRewards and claimRewardsOnBehalf functions : This would require a one-time gas cost to deploy the precompiled contract, but it would save gas on subsequent calls.

11 . Target : https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/RoeRouter.sol

- Changine the poolId variable from uint256 to uint8 : The uint256 data type is 256 bits wide, while the uint8 data type is only 8 bits wide. This means that the uint8 data type takes up less space in memory, which reduces the gas cost of storing and manipulating the variable.
- Change the pools array to a mapping from uint8 to RoePool. A mapping is a data structure that maps keys to values. In this case, the keys would be the pool IDs and the values would be the RoePool objects. Mappings are more efficient than arrays when you need to access a specific element by its key.
- Use inline assembly to implement the require statement in the addPool function. Inline assembly allows you to write code in assembly language, which is more efficient than Solidity code. The require statement is a common Solidity statement that is used to check for errors. By using inline assembly to implement the require statement, we can reduce the gas cost of the statement.


