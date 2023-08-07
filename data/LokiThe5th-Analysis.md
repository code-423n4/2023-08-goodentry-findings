## Approach  
An initial review of the documentation and automated findings was done to get a sense of the protocol. Thereafter the `GeVault.sol` and `TokenisableRange.sol` contracts were reviewed.  

Due to time constraints the focus of the security review was on the two aforementioned contracts. There was some difficulty with getting the project's Brownie-test framework to function on my setup, so the approach was to isolate code when a vulnerability was suspected and test it in Foundry or to simply rubber duck a scenario.

## Things learnt  
The project's use of Uniswap V3 Pools' concentrated liquidity provision (or ticks) as a way to approximate options-like functionality was a novel concept. Experience was also gained in Aave Lending Pools and the way in which this was integrated to allow leverage of positions using provided margin.  

## Comments  
The team noted at the start that the test coverage was not 100%, but it was also observed during review that tests were done on Mainnet Ethereum forks. The intended deployments were not adequately tested for the target chain and context, which led to a potential overflow issue in the `poolMatchesOracle()` function of the `GeVault.sol` contract on certain token configurations. 

It was also noted that there are contracts for this project in production on Arbitrum that are referenced in the documentation which are not verified, which complicates some reviews.

### Time spent:
20 hours