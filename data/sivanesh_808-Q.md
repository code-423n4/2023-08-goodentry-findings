**Report-1: Zero Address Check in Constructor of V3Proxy.sol**

**Introduction:**
In the smart contract [`V3Proxy.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L75), a constructor is defined to initialize the contract with certain parameters. The constructor takes three parameters: `_router`, `_quoter`, and `_fee`. This report focuses on the necessity of implementing a zero address check for these parameters in the constructor.

**Description:**
A crucial aspect of smart contract security is ensuring that inputs and parameters are properly validated to prevent unexpected behavior or vulnerabilities. The constructor provided in [`V3Proxy.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L75) initializes the contract's internal state with the `_router`, `_quoter`, and `_fee` parameters. However, it is important to ensure that these parameters are not set to the Ethereum zero address (0x0000000000000000000000000000000000000000).

The Ethereum zero address represents an invalid or unset address. Allowing the zero address as a parameter can lead to unexpected issues, vulnerabilities, or even loss of funds, as contracts might behave in unintended ways when interacting with invalid addresses.

**Recommendation:**
To enhance the security and robustness of the [`V3Proxy.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/helper/V3Proxy.sol#L75) contract, it is strongly recommended to include a zero address check for the `_router`, `_quoter`, and `_fee` parameters in the constructor. This check should ensure that none of these parameters are set to the Ethereum zero address before proceeding with the contract initialization.

**Example Zero Address Check:**
Here is an example of how the zero address check can be implemented within the constructor:

```solidity
constructor(ISwapRouter _router, IQuoter _quoter, uint24 _fee) {
    require(_router != ISwapRouter(address(0)), "Router address cannot be zero");
    require(_quoter != IQuoter(address(0)), "Quoter address cannot be zero");
    require(_fee > 0, "Fee should be greater than zero");

    ROUTER = _router;
    QUOTER = _quoter;
    feeTier = _fee;
}
```


**Report-2: Missed-Hardcoded Address in TokenisableRange Contract**

**Description:**
In the smart contract named `TokenisableRange`, a hardcoded Ethereum address is utilized. The address is assigned to the public constant variable `POS_MGR` and is directly defined within the contract code. The hardcoded address in question is `0xC36442b4a4522E871399CD717aBDD847Ab11FE88`, and it is assigned using the `INonfungiblePositionManager` interface.

**Address Hardcoded:**
```solidity
INonfungiblePositionManager constant public POS_MGR = INonfungiblePositionManager(0xC36442b4a4522E871399CD717aBDD847Ab11FE88);
```

**Location of the Hardcoded Address:**
You can find the hardcoded address in the `TokenisableRange` contract at [this specific line](https://github.com/code-423n4/2023-08-goodentry/blob/main/contracts/TokenisableRange.sol#L58C1-L58C1) within the contract code.

This practice of hardcoding addresses can introduce risks and inflexibility into the contract, as changing the target address may necessitate a redeployment of the entire contract. It's generally recommended to store external contract addresses as state variables or provide them as constructor arguments, enabling more flexibility in case the address needs to be updated in the future without needing to redeploy the contract.

It's advised to consider modifying the contract to enhance its upgradeability and maintainability by avoiding hardcoded addresses and introducing more flexible methods for specifying contract dependencies.

Please note that this assessment is based on the code as of the provided link and may not reflect changes made after that point. It's recommended to review the contract and its codebase for any updates or changes that may have occurred since the provided link.


**Report-3:Missing @Dev Emitted Event**

**Description:**

In the contract [`ICreditDelegationToken.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ICreditDelegationToken.sol#L5C3-L10C5), there is a missing `@dev` comment for the emitted event `BorrowAllowanceDelegated`. This event is declared as follows:

```solidity
event BorrowAllowanceDelegated(
    address indexed fromUser,
    address indexed toUser,
    address asset,
    uint256 amount
);
```

**Link to the Code:**

The declaration of the event can be found in the contract source code [here](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ICreditDelegationToken.sol#L5C3-L10C5).

**Recommendation:**

It is recommended to provide a descriptive `@dev` comment for the `BorrowAllowanceDelegated` event. This comment should explain the purpose and significance of the event, its parameters, and any relevant information for developers who might be interacting with the contract. Adding this comment will enhance the readability of the code and provide valuable context for anyone working with the contract.

**Example:**

Here's an example of how the missing `@dev` comment could be added:

```solidity
/**
 * @dev Emitted when the borrowing allowance is delegated from one user to another.
 *
 * This event indicates that the borrowing allowance for a specific asset has been transferred
 * from the `fromUser` to the `toUser` in the amount specified by `amount`.
 *
 * @param fromUser The address of the user delegating the borrowing allowance.
 * @param toUser The address of the user receiving the delegated borrowing allowance.
 * @param asset The address of the asset for which the borrowing allowance is being delegated.
 * @param amount The amount of borrowing allowance being delegated.
 */
event BorrowAllowanceDelegated(
    address indexed fromUser,
    address indexed toUser,
    address asset,
    uint256 amount
);
```

Adding this kind of comment helps other developers understand the purpose and behavior of the event without having to dive into the event's implementation details.

**Report-4: Pragma `abi`**  

It has come to our attention that the usage of `pragma abicoder v2` might lead to compatibility issues when using Solidity version 0.7.5. This report aims to outline the specific instances where this incompatibility has been identified and provide more information about the affected contracts.

**Description:**

The `pragma abicoder v2` directive is used to specify the default behavior of function parameter encoding and decoding in Solidity. It was introduced to improve the handling of function parameters and return values in more complex cases, particularly involving structs and arrays.

However, the usage of `pragma abicoder v2` appears to be problematic when used with Solidity version 0.7.5. This pragma version might not be fully supported or compatible with the older compiler version, leading to potential compilation errors or unexpected behavior.

**Affected Contracts:**

The following contracts have been identified as utilizing `pragma abicoder v2` and might face compatibility issues when used with Solidity 0.7.5:

1. [`IPoolInitializer.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IPoolInitializer.sol#L3C1-L3C20)
2. [`ISwapRouter.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ISwapRouter.sol#L3)
3. [`IAaveIncentivesController.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L3)
4. [`ILendingPoolConfigurator.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ILendingPoolConfigurator.sol#L3)
5. [`INonfungiblePositionManager.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L3)
6. [`IAaveLendingPoolV2.sol`](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveLendingPoolV2.sol#L3)

**Recommendation:**

To mitigate these compatibility issues, it is recommended to consider the following steps:

1. **Upgrade Compiler Version:** Consider upgrading the Solidity compiler version to a more recent one that better supports `pragma abicoder v2`, or use a version that is known to work well with this pragma version.

2. **Check Documentation:** Consult the official Solidity documentation and release notes to understand the compatibility matrix between `pragma abicoder v2` and various compiler versions. This can help in making informed decisions regarding the compiler version to be used.

3. **Code Review and Testing:** Thoroughly review and test the contracts that utilize `pragma abicoder v2` with the specific compiler version you intend to use. Identify and address any compilation errors or unexpected behavior that might arise due to the pragma version.

4. **Consider Alternative Solutions:** If compatibility issues persist, consider revisiting the usage of `pragma abicoder v2` and evaluate whether alternative approaches or workarounds can achieve the desired functionality without encountering compatibility problems.

It's important to stay updated with the latest advancements in the Solidity ecosystem and ensure that the chosen compiler version and pragmas are well-supported to maintain the reliability and security of your smart contracts.


**Report-5: Unused EVENT**

This report identifies and documents instances of unused events in the codebase of the "GoodEntry" project. Unused events can bloat the code and increase complexity, so it's important to clean them up for better code maintainability and readability.

Here are the identified instances of unused events along with their descriptions and corresponding links to the code:

1. **Event**: `event Transfer(address indexed from, address indexed to, uint256 value);`
   **Description**: This is a standard ERC-20 event for token transfers. It's declared in [IERC20.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IERC20.sol#L71) but not used within the project.

2. **Event**: `event Approval(address indexed owner, address indexed spender, uint256 value);`
   **Description**: Another standard ERC-20 event for approval of token spending. It's declared in [IERC20.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IERC20.sol#L77) but remains unused in the codebase.

3. **Event**: `event Approval(address indexed owner, address indexed spender, uint256 value);`
   **Description**: Similar to the above, this event is also declared in [IAToken.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAToken.sol#L21), but it's unused within the project.

4. **Event**: `event Mint();`
   **Description**: This event is declared in [IInitializableAToken.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IInitializableAToken.sol#L24) but not utilized within the codebase.

5. **Event**: `event Flash(....);`
   **Description**: This event is declared in [IUniswapV3Factory.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV3Factory.sol#L10C23-L10C23) but not used within the project.

6. **Event**: `event PoolCreated(address indexed token0, address indexed token1, ...);`
   **Description**: This event is declared in [IUniswapV3Factory.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV3Factory.sol#L29), but its declaration remains unused.

7. **Event**: `event AddressUpdated(string indexed name, address indexed newAddress);`
   **Description**: This event is declared in [ILendingPoolAddressesProvider.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ILendingPoolAddressesProvider.sol#L2), but it's not used within the codebase.

8. **Event**: `event MintOnDeposit(address indexed _to, uint256 _amount);`
   **Description**: This event is declared in [IAToken.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAToken.sol#L43) but remains unused within the project.

9. **Event**: `event Redeem(address indexed _from, address indexed _to, ...);`
   **Description**: This event is declared in [IAToken.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAToken.sol#L52) but is not utilized within the codebase.

10. **Event**: `event Transfer(address indexed sender, address indexed recipient, ...);`
    **Description**: This event is declared in [IUniswapV2Pair.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV2Pair.sol#L4-L5), but its declaration is unused.

11. **Event**: `event Mint(address to, uint256 tokenId, ...);`
    **Description**: This event is declared in [IUniswapV2Pair.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IUniswapV2Pair.sol#L24-L34), but it's not utilized within the codebase.

12. **Event**: `event Claimed(...);`
    **Description**: This event is declared in [IAaveIncentivesController.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/IAaveIncentivesController.sol#L6-L17), but its declaration remains unused.

13. **Event**: `event LendingPoolConfiguratorUpdated(...);`
    **Description**: This event is declared in [ILendingPoolConfigurator.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/ILendingPoolConfigurator.sol#L7-L18), but it's not used within the codebase.

14. **Event**: `event Transfer(uint256 indexed tokenId, address indexed from, ...);`
    **Description**: This event is declared in [INonfungiblePositionManager.sol](https://github.com/code-423n4/2023-08-goodentry/blob/main/interfaces/INonfungiblePositionManager.sol#L31-L44), but its declaration is unused.

These unused event declarations should be reviewed and potentially removed from the codebase to enhance code clarity and maintainability.