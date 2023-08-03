- Approach taken in evaluating the codebase.

I followed this approach
Critical Contracts - Critical Functions - Critical Parts

I looked out for critical parts such as external calls to other contracts (.claimRewards(), .withdraw(), .executeOptions()) to prevent Reentrancy Attacks.

I also looked out for critical functions that needed authorization and lacked it to prevent Access Control Attack.

- Anything that you learned from evaluating the codebase

Organizing a contract's code into sections will make it more readable and easy to navigate  (E.g. Internal functions grouped separately from external ones, etc.) 

Admin and critical functions should be protected from Access control Attacks using the onlyOwner modifier .

- Any comments for the judge to contextualize your findings
I included permalinks as instructed to help you locate the functions quickly. 

### Time spent:
35 hours