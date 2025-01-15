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
The Prodigal Contract vulnerability was first identified and introduced as a concept in the research presented in the MAIAN tool [1].  The MAIAN tool highlighted this as one of the critical vulnerabilities in Ethereum smart contracts, which can lead to the unintentional loss of funds. A sample contract demonstrating this vulnerability, titled example_prodigal.sol, is available in the MAIAN repository [2]
### Vulnerability Points
The Adoption Contract demonstrates several critical flaws that, when exploited, can result in the unintended transfer of Ether to unauthorized or arbitrary addresses. Below, we outline the key vulnerability points, referencing specific sections of the contract:
- Unrestricted Access to Transfer Functions
```Solidity
function returnEth(address oldOwner, uint256 price) public payable {
    oldOwner.transfer(price);
}
```
The returnEth function is publicly accessible and allows any user to call it with arbitrary parameters. There are no checks to verify if the caller is authorized or if the oldOwner address is valid. A malicious user can call this function and pass an arbitrary address as oldOwner, causing Ether to be transferred to the attacker's address without justification. This violates the principles of secure access control and accountability.
- Lack of Validation on Inputs
```Solidity
function adopt(uint pepeId) public payable returns (uint, uint) {
    require(pepeId >= 0 && pepeId <= 15);
    require(msg.value >= data[pepeId].price * uint256(1));
    returnEth(data[pepeId].owner, (data[pepeId].price / 10) * (9));
    gimmeTendies(ceoAddress, (data[pepeId].price / 10) * (1));
    data[pepeId].owner = msg.sender;
    return (pepeId, data[pepeId].price);
}
```
The adopt function validates the pepeId range but fails to ensure that the data[pepeId].owner and data[pepeId].price fields are correctly initialized or valid. If the data[pepeId].owner is set to an invalid or malicious address (e.g., through a prior manipulation of the contract’s state), Ether will be transferred to that address during execution of returnEth.
-  Unsafe Ether Transfer Mechanisms
```Solidity
oldOwner.transfer(price);
```
The contract uses the .transfer method to send Ether. While .transfer is often considered safe, it can fail under certain conditions, such as:
If the recipient contract consumes excessive gas in its fallback function. If the recipient contract is non-payable. Failed transfers due to gas limitations or recipient-side issues can disrupt the contract’s intended logic, leaving funds unrecoverable.
 
### Combined Impact
The combination of these vulnerabilities creates a scenario where:
Unintended Transfers: Funds are sent to arbitrary or unauthorized addresses due to a lack of input validation and access control.

Disruption of Logic: Failed transfers can interrupt the contract’s intended functionality, potentially leaving it in an inconsistent state.

## References
[1] Nikolić, I., Kolluri, A., Sergey, I., Saxena, P., & Hobor, A. (2018, December). Finding the greedy, prodigal, and suicidal contracts at scale. In Proceedings of the 34th annual computer security applications conference (pp. 653-663).

[2] https://github.com/ivicanikolicsg/MAIAN
