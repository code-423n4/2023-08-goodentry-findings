# Disclaimer

Hello, first report here. Sorry for the poor formatting and if the findings are not appropriate.

# Findings Summary
[NC-01] | Usage of block timestamp 
[NC-02] | Same function duplicated

# Detailed Findings

# [NC-01] Usage of block timestamp
block.timestamp can be influenced by miners to a certain degree.

# Recommendation:
Use block.number instead of block.timestamp or now to reduce the risk of
MEV attacks. Check if the timescale of the project occurs across years,
days and months rather than seconds. If possible, it is recommended to
use Oracles

# [NC - 02] Same function duplicated

The sqrt function is declared and used in TokenisableRange.sol and LPOracle.sol
The sqrt function in the Sqrt.sol library is not used in these two cases.

# Recommendation
Using the Sqrt.sol library to reduce code redundancy

### Time spent:
8 hours