# Erroneous Visibility
This vulnerability was first discovered in an attack on the Parity wallet [1], occurring when a function’s visibility is improperly defined, enabling unauthorized access. Solidity provides four visibility types to control access to functions: Public (accessible from anywhere), External (callable only externally), Internal (accessible within the contract and its derived contracts), and Private (restricted to the defining contract). Functions not intended for external calls must be declared as Private or Internal, but Solidity defaults to Public, creating opportunities for exploitation. From version 0.5.0 onward, Solidity mitigates this by requiring explicit visibility declarations [39]. However, the issue persists if developers fail to correctly define function visibility.

## Toy Example
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
On July 19, 2017, a critical vulnerability in the Parity Wallet led to one of the most significant exploits in blockchain history, resulting in the loss of $31,000,000 worth of Ether. The issue arose due to a fundamental design flaw in the multi-signature wallet contract, specifically in the use of a shared library for wallet initialization and state management. The vulnerability allowed an attacker to reinitialize the wallet and set themselves as the owner, enabling unauthorized access to the funds. The vulnerability is highlighted in the following block of code, where the fallback function delegates unknown method calls to a shared library:
```Solidity
function() payable {
    if (msg.value > 0) {
        Deposit(msg.sender, msg.value);
    } else if (msg.data.length > 0) {
        _walletLibrary.delegatecall(msg.data);  
    }
}
```
### How the Exploit Works
- Exploiting the Fallback Function: The fallback function forwards any unknown method call to the _walletLibrary using delegatecall, executing the library's code in the context of the current wallet contract.
- Reinitializing the Wallet: The attacker calls the initWallet() function from the shared library via the fallback. This function reinitializes the wallet and allows the attacker to set new owners.
 ```Solidity
function initWallet(address[] _owners, uint _required, uint _daylimit) {
    initDaylimit(_daylimit);
    initMultiowned(_owners, _required);
}
```
- Overwriting Ownership: The initMultiowned() function overwrites the ownership details without verifying if the wallet was already initialized.
```solidity
function initMultiowned(address[] _owners, uint _required) {
    m_numOwners = _owners.length + 1;
    m_owners[1] = uint(msg.sender);
    m_ownerIndex[uint(msg.sender)] = 1;
    for (uint i = 0; i < _owners.length; ++i) {
        m_owners[2 + i] = uint(_owners[i]);
        m_ownerIndex[uint(_owners[i])] = 2 + i;
    }
    m_required = _required;
}
```

  ### Root Cause of the Exploit
- The initWallet and initMultiowned functions in the library were not marked as internal or private, allowing them to be invoked externally.
- The fallback function’s reliance on delegatecall without restricting accessible methods enabled the attacker to exploit the vulnerability.
- There was no mechanism to verify whether the wallet had already been initialized, allowing the attacker to overwrite the ownership structure

  ## Refwrences
  [1] https://medium.freecodecamp.org/a-hacker-stole-31m-of-ether-how-it-happened-and-what-it-means-forethereum-9e5dc29e33ce.
 




