# Dangerous Delegatecall
This vulnerability was initially identified during an attack on the Parity wallet, leveraging the delegate-call opcode in the EVM. The delegate-call opcode permits the CoB of a callee contract to be integrated into the CoB of the calling contract, facilitating code reuse. 
This might lead to a malicious callee contract gain direct access to manipulate the state variables of the calling contract.

## Toy Example
```Solidity
// Library contract that contains a function to set a value
contract Library {
    uint public value; // Public variable to store the value

    // Function to set the value
    function setValue(uint _value) public {
        value = _value; // Update the stored value
    }
}

// Attacker contract that exploits delegatecall
contract Attacker {
    uint public value; // Public variable to store the attacker's value

    // Function to perform the attack using delegatecall
    function attack(address _target, uint _value) public {
        // Use delegatecall to execute setValue in the target contract
        (bool success, ) = _target.delegatecall(abi.encodeWithSignature("setValue(uint256)", _value));
        require(success, "Attack failed!"); // Ensure the call was successful
    }
}
```

## Real-World Example
In December 2023, a critical vulnerability was identified in smart contracts combining the **ERC-2771Context** standard and the **Multicall** function. The issue stemmed from the interaction between these two features, where improper handling of the `delegatecall` opcode allowed attackers to manipulate the `_msgSender()` value, leading to arbitrary address spoofing.

The vulnerability exploited the following sequence of events:
1. **ERC-2771 Context Forwarding**: The ERC-2771 standard enables meta-transactions by allowing calls to be forwarded through a trusted relayer, which modifies the calldata to include the original sender's address.
2. **Multicall Functionality**: The Multicall function allows multiple operations to be executed in a single transaction, often leveraging `delegatecall` for efficiency.
3. **Delegatecall Context Retention**: The `delegatecall` opcode executes code from a target contract in the context of the calling contract, retaining the caller’s storage and `msg.sender`.

In vulnerable implementations, the `_msgSender()` function, which resolves the transaction's sender, was manipulated during the processing of forwarded requests. Attackers wrapped malicious calldata within meta-transactions and used `delegatecall` to retain their spoofed address. This allowed them to bypass permission checks or impersonate other users.

Attackers leveraged this vulnerability to execute unauthorized actions in affected contracts:
- In one instance, an attacker impersonated a user to transfer 84.59 ETH.
- In another, 17,394 USDC was stolen by exploiting the same flaw.



## References

[1] https://eips.ethereum.org/EIPS/eip-2771

[2] https://blog.openzeppelin.com/arbitrary-address-spoofing-vulnerability-erc2771context-multicall-public-disclosure

[3] https://blog.solidityscan.com/unveiling-the-erc-2771context-and-multicall-vulnerability-f96ffa5b499f
