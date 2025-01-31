# Memory Access Error
When software tries to read from or write to a memory location that it is not allowed to access, memory access problems happen. These issues often arise from improper memory management or unchecked operations, leading to unintended interactions with or corruption of data in restricted memory locations. Unexpected behavior, failures, or critical security vulnerabilities could result from such issue. Memory access problems can manifest in various forms, including Array Out-of-Bounds Access, Buffer Overflow, Segmentation Faults, Use-After-Free, and Null Pointer Dereference. Each of these highlights a specific way improper memory access can lead to failures or vulnerabilities.
## Toy Example
```Solidity
pragma solidity ^0.8.0;

contract Wallet {
    address private owner;  
    uint[] private bonusCodes;  

    constructor() {
        owner = msg.sender;  
    }

    function PopCode() public {
        require(0 <= bonusCodes.length, "Invalid length");
        @r bonusCodes.length--;@  // Direct length modification, potential underflow
    }

    function SetCodeAt(uint idx, uint c) public {
        require(idx < bonusCodes.length, "Index out of bounds");
        @o minimize(|&(bonusCodes[idx]) - 0xffcaffee|);@ // Logical flaw in minimize logic
        bonusCodes[idx] = c; // Update value at index
    }

    function Destroy() public {
        require(msg.sender == owner, "Only owner can destroy contract");
        @g selfdestruct(msg.sender);@  // Ownership misuse risk
    }
}
```
## Real World Example
```Solidity
// Vulnerable Smart Contract
pragma solidity ^0.8.0;

contract Auction {
    address public highestBidder;
    uint public highestBid;

    mapping(address => uint) public bids;

    function bid() external payable {
        require(msg.value > highestBid, "Bid too low");

        // Returning funds to the previous highest bidder
        if (highestBidder != address(0)) {
            payable(highestBidder).transfer(highestBid); // 🔴 DoS Risk: If this fails, whole function fails
        }

        highestBidder = msg.sender;
        highestBid = msg.value;
    }
}

```
## References
