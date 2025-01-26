
## Reentrancy

The reentrancy vulnerability occurs when functions that are not intended to allow repeated calls are exploited, often through fallback functions. This enables attackers to bypass critical checks and drain the contract of Ether or deplete the transaction's gas. The vulnerability arises when a contract relies on state variables that should be updated before making external calls but are not, and when no gas limit is imposed on transferring control to another contract. A notorious example of this vulnerability being exploited is the "DAO" attack, where attackers used it to steal a significant amount of Ether.




## Toy Example
```Solidity
pragma solidity ^0.8.0;

contract ReentrancyExample {
    mapping(address => uint) public balance;

    function deposit() public payable {
        balance[msg.sender] += msg.value;
    }

    function withdraw(uint _amount) public {
        require(balance[msg.sender] >= _amount, "Insufficient balance");

        (bool success, ) = msg.sender.call{value: _amount}(""); // Vulnerable line: susceptible to reentrancy attack
        require(success, "Transfer failed");

        // Update state AFTER external call (vulnerable)
        balance[msg.sender] -= _amount;
    }
}

```

## Real-World Example

On November 11, 2024, a critical reentrancy vulnerability was exploited in the SmartLoan contract, leading to a financial loss of $4.75 million. The vulnerability arose from an improper external call using unverified input within the contract's logic. Specifically, the use of the call method in the line [1]:

 ```Solidity
address(SmartLoan).call(claimRewardData);

```
The exploit occurred when an attacker deployed a malicious contract designed to interact with the vulnerable SmartLoan contract. By using the call method, the attacker triggered reentrant calls through the malicious contract, allowing them to manipulate the state of SmartLoan mid-execution and bypass critical validations such as balance and collateral checks. This attack was further amplified using a flash loan from Balancer, enabling the attacker to drain significant assets from the protocol.

## References

[1] https://github.com/SunWeb3Sec/DeFiHackLabs
