# Table of Contents
| Number | Issues Details                                                                                         | Count |
| :----- | :----------------------------------------------------------------------------------------------------- | :---- |
| [L-1] | Including Critical Parameters in Events | 3 |
| [L-2] | Utilize `_safeMint` for Safer Minting | 1 |
| [L-3] | Beware of Loss of Funds with `payable.transfer` | 1 |
| [L-4] | Implementing Maximum Fee Limit Instead of Allowing `100%` Fees | 1 |
| [N-1] | Repeated File Imports | 2 |
| [N-2] | Advocate for Distinct Files for Interface Declarations | 2 |
| [N-3] | Variations in Comment Spacing | 1 |
| [N-4] | Limit Usage of All-Capitalized Variable Names to `constant`/`immutable` Variables | 2 |
| [N-5] | Indicating Interfaces with `I` Prefix in Contract Names | 2 |
| [N-6] | Any duplicated `require()` or `revert()` checks should be restructured into a modifier or function for better efficiency and readability | 1 |
| [N-7] | Either remove empty blocks or have them emit an event, as they serve no purpose as is | 1 |
| [N-8] | Functions that are certain to revert when invoked by regular users can be designated as `payable` | 1 |

## [L-1]</a><a name="L-1"> Including Critical Parameters in Events

To ensure the usability and effectiveness of events, it is crucial to include all the necessary parameters, including the source address (`msg.sender`), when emitting an event. By omitting critical parameters, the event loses its intended purpose as a means for contracts or web pages to react to user actions.

<details>
<summary><i>There are 3 instances of this issue:</i></summary>

```solidity
File: OptionsPositionManager.sol
emit LiquidatePosition(user, debtAsset, debt, amt0 - amts[0], amt1 - amts[1]);
```

```solidity
File: TokenisableRange.sol
emit InitTR(address(asset0), address(asset1), startX10, endX10);
```

```solidity
File: OptionsPositionManager.sol
emit ClosePosition(user, debtAsset, debt, amt0, amt1);
```
</details>



## [L-2]</a><a name="L-2"> Utilize `_safeMint` for Safer Minting

To enhance the safety and security of the minting process in your contract, it is recommended to use the `_safeMint` function instead of `_mint`. The `_safeMint` function incorporates additional checks and safeguards to mitigate potential risks.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: GeVault.sol
_mint(msg.sender, liquidity);
```
</details>



## [L-3]</a><a name="L-3"> Beware of Loss of Funds with `payable.transfer`

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: GeVault.sol
payable(msg.sender).transfer(bal);
```
</details>



## [L-4]</a><a name="L-4"> Implementing Maximum Fee Limit Instead of Allowing `100%` Fees

To mitigate potential risks and maintain a balanced fee structure, it is advisable to disallow the setting of fees to `100%` altogether. Allowing fees to be set at such a high percentage poses a considerable risk and may have unintended consequences.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: GeVault.sol
function setBaseFee(
```

</details>



## [N-1]</a><a name="N-1"> Repeated File Imports

Numerous instances have been observed where the same file is imported multiple times.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: RangeManager.sol
import "./openzeppelin-solidity/contracts/token/ERC20/utils/SafeERC20.sol";
```

```solidity
File: TokenisableRange.sol
import "../interfaces/IAaveOracle.sol";
```
</details>



## [N-2]</a><a name="N-2"> Advocate for Distinct Files for Interface Declarations

It's good practice to place interface declarations in a distinct file, rather than housing them within the same file as the contract.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: LPOracle.sol
interface UniswapV2Pair {
```

```solidity
File: LPOracle.sol
interface IERC20 {
```

```solidity
File: V3Proxy.sol
interface ISwapRouter {
```

```solidity
File: V3Proxy.sol
interface IQuoter {
```

```solidity
File: V3Proxy.sol
interface IWETH9 is IERC20 {
```
</details>



## [N-3]</a><a name="N-3"> Variations in Comment Spacing

Certain lines exhibit inconsistent spacing in comments. This issue is observed where some lines use "// x" while others use "//x". The examples provided below highlight instances that deviate from the prevailing pattern within each file.

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: OptionsPositionManager.sol
{ //localize vars
```
</details>



## [N-4]</a><a name="N-4"> Limit Usage of All-Capitalized Variable Names to `constant`/`immutable` Variables

Restrict the application of fully capitalized variable names to `constant` or `immutable` variables only. In cases where the variable's value is dependent on its originating class, utilizing a `view`/`pure` function is more suitable, as shown in this <a href="https://github.com/OpenZeppelin/openzeppelin-contracts/blob/76eee35971c2541585e05cbf258510dda7b2fbc6/contracts/token/ERC20/extensions/draft-IERC20Permit.sol#L59">example</a>.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: PositionManager.sol
ILendingPoolAddressesProvider public ADDRESSES_PROVIDER; // IFlashLoanReceiver  requirement
```

```solidity
File: PositionManager.sol
ILendingPool public LENDING_POOL; // IFlashLoanReceiver  requirement
```

```solidity
File: PositionManager.sol
RoeRouter public ROEROUTER;
```

```solidity
File: GeVault.sol
IWETH public WETH;
```
</details>



## [N-5]</a><a name="N-5"> Indicating Interfaces with `I` Prefix in Contract Names

To maintain consistency and improve code readability, it is advised to prefix interface names with the letter `I` in contracts.

<details>
<summary><i>There are 2 instances of this issue:</i></summary>

```solidity
File: LPOracle.sol
interface UniswapV2Pair {
```

```solidity
File: OptionsPositionManager.sol
interface  AmountsRouter {
```
</details>



## [N-6]</a><a name="N-6"> Any duplicated `require()` or `revert()` checks should be restructured into a modifier or function for better efficiency and readability

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: V3Proxy.sol
require(path.length == 2, "Direct swap only");
require(path.length == 2, "Direct swap only");
require(path.length == 2, "Direct swap only");
require(path.length == 2, "Direct swap only");
require(path.length == 2, "Direct swap only");
require(path.length == 2, "Direct swap only");
require(path.length == 2, "Direct swap only");
require(path.length == 2, "Direct swap only");
```
</details>



## [N-7]</a><a name="N-7"> Either remove empty blocks or have them emit an event, as they serve no purpose as is

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: OptionsPositionManager.sol
constructor (address roerouter) PositionManager(roerouter) {}
```
</details>



## [N-8]</a><a name="N-8"> Functions that are certain to revert when invoked by regular users can be designated as `payable`

<details>
<summary><i>There are 1 instances of this issue:</i></summary>

```solidity
File: V3Proxy.sol
function emergencyWithdraw(ERC20 token) onlyOwner external {
```
</details>