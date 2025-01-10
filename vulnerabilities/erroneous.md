# Erroneous
This vulnerability was first discovered in an attack on the Parity wallet [1], occurring when a function’s visibility is improperly defined, enabling unauthorized access. Solidity provides four visibility types to control access to functions: Public (accessible from anywhere), External (callable only externally), Internal (accessible within the contract and its derived contracts), and Private (restricted to the defining contract). Functions not intended for external calls must be declared as Private or Internal, but Solidity defaults to Public, creating opportunities for exploitation. From version 0.5.0 onward, Solidity mitigates this by requiring explicit visibility declarations [39]. However, the issue persists if developers fail to correctly define function visibility.








## Toye Example
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.4.24; // Older version where this vulnerability exists

contract VulnerableContract {
    // This function should be private or internal, but due to the default, it is public
    function changeOwner(address newOwner) public {
        owner = newOwner; // 🔴 Vulnerability: Attackers can call this function externally and change the owner
    }

    address public owner;

    constructor() public {
        owner = msg.sender; // Initial owner is set to the deployer of the contract
    }
}

```

## Real World Example
## References
[1] https://medium.freecodecamp.org/a-hacker-stole-31m-of-ether-how-it-happened-and-what-it-means-forethereum-
9e5dc29e33ce.

