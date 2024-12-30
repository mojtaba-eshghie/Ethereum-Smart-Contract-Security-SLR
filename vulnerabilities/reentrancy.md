## Description

The reentrancy vulnerability occurs when certain functions, unintentionally designed to be non-reentrant, are maliciously called reentrant (e.g., through fallback routines). This allows the attacker to bypass the necessary validity checks until the caller contract is drained of Ether or the transaction runs out of gas. The vulnerability arises due to two main reasons: first, a contract's control flow decision relies on some of its state variables that should be updated by the contract itself, but are not, before calling another contract; and second, there is no gas limit when handing over the control flow to another contract. The "DAO" attack exploited this vulnerability directly.




## Toy Example
pragma solidity ^0.8.0;

contract SecureContract {
    mapping(address => uint) public balance;

    // Function to deposit Ether into the contract
    function deposit() public payable {
        balance[msg.sender] += msg.value;
    }

    // Secure withdraw function
    function withdraw(uint _amount) public {
        require(balance[msg.sender] >= _amount, "Insufficient balance");

        // Update state first to prevent reentrancy
        balance[msg.sender] -= _amount;

        // External call to send Ether
        (bool success, ) = msg.sender.call{value: _amount}("");
        require(success, "Transfer failed");
    }
}



## Real-World Example






## ... 
