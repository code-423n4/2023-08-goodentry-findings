## Separate Treasury References in Multiple Contracts

### Description

GeVault and RoeRouter have separate references to the treasury address. This design approach can lead to potential issues and inconsistencies in the system.

### Problem

1. **Lack of Centralized Control**: Having separate treasury references in multiple contracts can result in a lack of centralized control over the treasury. Each contract may have its own implementation and logic for interacting with the treasury, increasing the complexity of managing funds and making it difficult to enforce consistent treasury operations.
2. **Inconsistent Treasury Operations**: With separate treasury references, there is a risk of inconsistent treasury operations across contracts. Each contract may handle treasury-related functions differently, leading to discrepancies in treasury management, such as fee collection, fund transfers, or updates to treasury parameters.
3. **Security Vulnerabilities**: If each contract has its own treasury reference, it becomes more challenging to ensure the security of the treasury. Inconsistencies in the implementation of treasury-related functions may introduce vulnerabilities, such as incorrect fund transfers or unauthorized access to the treasury.
4. **Maintenance Challenges**: Having separate treasury references can make it challenging to update or modify treasury-related functionality across multiple contracts. Any changes to treasury operations would require updating each contract individually, increasing maintenance efforts and the potential for errors.

### Recommendation

To address the issues mentioned above, it is recommended to centralize the treasury reference in a single contract or library. This centralized contract can serve as the single source of truth for treasury operations and ensure consistent and secure management of funds. Contracts that require access to the treasury can then interact with the centralized treasury contract through well-defined and standardized functions, ensuring consistency and reducing the risk of security vulnerabilities.

By centralizing the treasury reference, the system can benefit from centralized control, improved consistency, enhanced security, and easier maintenance of treasury-related operations.