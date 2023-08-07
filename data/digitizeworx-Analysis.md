## Analysis Report

GoodEntry is a perpetual options trading platform that is built on top of Uniswap v3. It uses single-tick liquidity to provide users with limited downside exposure. The protocol consists of three main components:

* Tokenized Uniswap v3 positions (TRs)
* ezVaults
* A lending pool

TRs are ERC-20 tokens that represent single-tick liquidity on Uniswap v3. They can be used to trade perps with limited downside. ezVaults are smart contracts that hold TRs and rebalance the underlying liquidity as needed. The lending pool is a forked version of Aave v2 that allows users to borrow and lend TRs.

The core idea behind GoodEntry is that single-tick liquidity in Uniswap behaves as a limit order, whose payout is similar to writing an option. Borrowing such liquidity and removing it from the tick gives a payout similar to buying an option.

The protocol has been audited by several security firms and is now under audit on the platform of Code4rena and it has a high level of code coverage. However, there are still some potential risks that should be considered before using the protocol.

### Centralization Risks

The lending pool is a centralized component of the protocol. This means that if the lending pool is hacked, it could lead to the loss of user funds. Additionally, the lending pool is controlled by a single entity, which could lead to censorship or other problems.

### Mechanism Review

The mechanism of the protocol is complex and there are some potential risks that should be considered. For example, if the price of the underlying asset moves significantly, it could lead to liquidation of user positions. Additionally, the protocol is not fully collateralized, which means that there is a risk of default.

### Systemic Risks

The protocol is designed to be a decentralized platform, but there are some potential systemic risks that should be considered. For example, if a large number of users were to sell their TRs at the same time, it could lead to a crash in the price of the protocol's tokens. Additionally, the protocol is dependent on the underlying liquidity of Uniswap v3, which could pose a risk if the liquidity of Uniswap v3 were to decrease.

GoodEntry is a promising project with a lot of potential. 

## Recommendations

* The lending pool should be decentralized to reduce the risk of centralization.
* The protocol should be fully collateralized to reduce the risk of default.
* The protocol should be stress-tested to identify and mitigate potential systemic risks.

## Conclusion

GoodEntry is a promising project with a lot of potential. However, there are some risks that should be considered before using the protocol. The protocol should be decentralized, fully collateralized, and stress-tested to mitigate the potential risks.

### Time spent:
31 hours