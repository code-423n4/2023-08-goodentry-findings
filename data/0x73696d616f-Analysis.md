## Comments for the judge to contextualize your findings
Migrated the setup to foundry to write better POCS. Reach out in case you want a repo with the POCS, so you can run them.

## Approach taken in evaluating the codebase
Skimmed through the docs and then took a deep dive in each smart contract individually.

## Architecture recommendations
The codebase is very well organized, don't have any specific recommendation.

## Codebase quality analysis
The codebase is quite mature, some fuzz testing should be added to complement it. There are a few security concerns, but most should be easy to fix.

## Centralization risk
The owner has a very previleged role and can pull the rug at any time. For instance, in the `TokanisableRange`, the `ticks` can be modified via `modifyTick()`, which then lets the owner deposit liquidity there an do whatever it pleases.

## Mechanism review
Good entry leverages UniswapV3 tick feature, using them as options. Essentially, if liquidity is deposited above the current price, it works as a buy option, enabling users to buy when the price is low. Contrarily, on the other side of the liquidity range, if it is placed below, it works as a sell option and assets can be bought when the price is low.

It splits the liquidity into several consecutive ticks, each one of them being a `TokanisableRange` to enable the different options. 

The liquidity is concentrated from in `GeVault`, which uses shares, the liquidity from Uniswap, so that each user has their balance represented.

The options are managed through the `OptionsPositionManager`, which allows buying (with leverage), selling, withdrawing, liquidating and closing debt. All this using both Uniswap and AaveV3 swap and flashloan, respectively, functionalities.

## Systemic risks
- AaveV3 exploits
- UniswapV3 exploits
- Oracle manipulation


### Time spent:
40 hours