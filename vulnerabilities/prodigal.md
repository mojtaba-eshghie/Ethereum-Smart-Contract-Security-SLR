# Prodigal
This vulnerability occurs when a smart contract improperly sends Ether to arbitrary addresses without adequately controlling the transfer process. It is commonly referred to in the literature as 'Leaking Ether to Arbitrary Addresses.' The vulnerability arises when a contract transfers Ether to addresses that are not its owners and have not meaningfully interacted with it such as not depositing Ether or providing critical data [].
## Toye Example
```Solidity
pragma solidity ^0.8.0;

contract ProdigalExample {
    address public owner;

    constructor() {
        owner = msg.sender; // Set the contract deployer as the owner
    }

    // Function to send Ether to any address
    function sendEther(address payable recipient, uint256 amount) public {
        // 🔴 Vulnerability: No validation on the recipient address
        recipient.transfer(amount); 
    }

    // Function to deposit Ether into the contract
    function deposit() public payable {}

    // Get the contract balance
    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}

```
## Real World Example


## References
