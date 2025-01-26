# Dangerous Balance Inequality
Using \verb|==| instead of $a \geq b$ when verifying inventory arises from the flawed assumption that the exact match will always occur. This practice can lead to a critical security vulnerability, as even minor, unexpected changes in inventory values whether intentional or accidental could bypass verification logic [1]. Such an oversight may enable unauthorized access or manipulation of sensitive data, ultimately compromising system integrity and exposing it to exploitation. According to [2], this vulnerability was first reported and identified by Slither's detectors.
## Toy Example
```Solidity
pragma solidity ^0.8.0;

contract TokenSale {
    uint public constant TOKEN_PRICE = 1 ether;  
    uint public constant FUND_GOAL = 100 ether;  
    uint public totalFunds;  
    mapping(address => uint) public balances; 

    function buyTokens() public payable {
        require(msg.value > 0, "Send ETH to buy tokens");  
        require(totalFunds + msg.value <= FUND_GOAL, "Sale goal reached");  

        uint tokens = msg.value / TOKEN_PRICE; // Calculate the number of tokens purchased
        balances[msg.sender] += tokens; // Update the buyer's token balance
        totalFunds += msg.value; // Update the total funds received
    }

    function saleFinished() public view returns (bool) {
        // Vulnerability: This condition only returns true when totalFunds is exactly equal to FUND_GOAL (100 ether).
        // If totalFunds exceeds or falls just short of FUND_GOAL due to rounding issues or edge cases,
        // the function will incorrectly return false, even though the intended goal has been achieved.
        return totalFunds == FUND_GOAL;
    }
}

```


## Real word Example
In 2019, a significant vulnerability was identified in a smart contract called Lockdrop, caused by the improper use of the strict equality operator (==). This contract was designed to lock funds and distribute new tokens, but a logical error led to a critical vulnerability that created a gridlock in the contract's functionality [3].
In the lock() function, a new contract named Lock was created, and a certain amount of Ether (msg.value) was sent to it. The following assertion was then used to check whether the balance of the Lock contract was exactly equal to msg.value:
```Solidity
assert(address(lockAddr).balance == msg.value);
```
This strict equality check (==) caused the assertion to fail if the balance of the Lock contract was not exactly equal to msg.value for any reason, resulting in the transaction being reverted.
An attacker could exploit this vulnerability by predicting the address of the new Lock contract (using Ethereum's deterministic address calculation) and sending a small amount of Ether to this address before the lock() function was executed. This would cause the initial balance of the Lock contract to be greater than zero, leading the assertion to fail since the balance would no longer match msg.value. Consequently, the contract's functionality would become stuck, as no one could successfully call the lock() function. This vulnerability had the potential to lock users' funds indefinitely and disrupt the overall functionality of the contract.


## References
[1] Demir, M., Alalfi, M., Turetken, O., & Ferworn, A. (2019, July). Security smells in smart contracts. In 2019 IEEE 19th International Conference on Software Quality, Reliability and Security Companion (QRS-C) (pp. 442-449). IEEE.

[2] https://blog.trailofbits.com/2019/07/03/avoiding-smart-contract-gridlock-with-slither/

[3] [https://github.com/crytic/slither/wiki/Detector-Documentation#dangerous-strict-equalities](https://github.com/crytic/slither/wiki/Detector-Documentation#dangerous-strict-equalities)

