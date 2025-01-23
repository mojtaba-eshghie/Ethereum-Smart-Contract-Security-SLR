# Call-Stack Depth Limit
Due to Ethereum's 1024-frame stack limit per transaction, attackers can use recursive calls to fill the stack and prevent following calls from succeeding. This is known as the Call-stack Depth issue. By raising the gas price for external calls, EIP-150 somewhat reduced the impact of these attacks, and EIP-214 reduced the dangers associated with external interactions by introducing STATICCALL for read-only calls. The stack depth limit, however, is still in effect. The EVM's architecture, which was created to avoid excessive resource usage and preserve system efficiency, is inherently limited. Network performance could be harmed by changing this restriction, and contracts with intricate or sequential calls are still vulnerable to this problem [1].

## Toy Example
```Solidity
pragma solidity ^0.4.19;

contract Vulnerable {
    address public previousUser;

    // Function to participate and return funds to the previous user
    function participate() public payable {
        require(msg.value > 0, "Send some ether to participate");

        // Refund ether to the previous user
        if (previousUser != address(0)) {
            previousUser.send(msg.value); // 🔴 Potential Vulnerability: If the previousUser is a malicious contract, it can trigger a fallback function that creates a recursive call and reaches the stack limit.
        }

        // Update the previous user
        previousUser = msg.sender;
    }
}

pragma solidity ^0.4.19;

contract Attacker {
    address targetContract;

    // Constructor to set the target contract address
    function Attacker(address _targetContract) public {
        targetContract = _targetContract;
    }

    function() public payable {
        targetContract.call.value(msg.value)(); // 🔴 Potential Vulnerability: The fallback function here makes recursive calls to the target contract, filling the stack depth to the maximum limit (1023 calls).
    }

    function attack() public payable {
        require(msg.value > 0, "Send some ether to start the attack");
        targetContract.call.value(msg.value)();
    }
}

```

## Real World Example

## References
[1] Ziyuan Li, Wangshu Guo, Quan Xu, Yingjie Xu, Huimei Wang, and Ming Xian. Research on blockchain smart contracts vulnerability and a code
audit tool based on matching rules. In Proceedings of the 2020 International Conference on Cyberspace Innovation of Advanced Technologies, pages
484–489, 2020.
