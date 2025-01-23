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
The Call Stack Attack in the GovernMental contract occurred before the implementation of EIP-150 and exploited the stack depth limit (1024 levels) in the Ethereum Virtual Machine (EVM). In this attack, the attacker used a malicious contract to create recursive calls, increasing the stack depth to 1023. This caused the next call at level 1024 to fail due to the limit. As a result, Ether transfers to intended addresses in the contract failed, and the jackpot was not paid to the recipients. Consequently, the contract was locked, its operations stopped, and 1100 Ether (valued at approximately over $1.5 million in 2023) remained inaccessible.
With the implementation of EIP-150, similar attacks have become economically impractical since the new gas limit for recursive calls has significantly increased the cost of executing such attacks. This incident underscores the importance of thoroughly reviewing smart contract code and adopting best design practices, such as verifying call results and splitting tasks into smaller arrays to ensure reliability and security. The following paragraphs will explain the contract code lines mentioned.
```Solidity
if (lastTimeOfNewCredit + TWELVE_HOURS < block.timestamp) {
    msg.sender.send(amount);
    credAddr[credAddr.length - 1].send(profitFromCrash);
    corruptElite.send(this.balance);
    lastPaid = 0;
    lastTimeOfNewCredit = block.timestamp;
    profitFromCrash = 0;
    credAddr = new address  credAmt = new uint ;
 und += 1;
    return false;
}
lies in the behavior of send and the stack depth limitation (Stack Depth) in Ethereum's EVM. The send function is used to transfer Ether to a specified address. If the destination address is a smart contract, the fallback function of that contract is automatically executed. An attacker can create a malicious contract that makes recursive calls in its fallback function. These recursive calls can increase the stack depth to 1023. When the send function is executed in the line

```Solidity
credAddr[credAddr.length - 1].send(profitFromCrash);

```
the stack depth reaches 1024, and due to the EVM's stack depth limitation, this call fails. This issue highlights a flaw in the contract's design, as it neglects to handle this limitation properly. As a result of the failed send call, the intended Ether is not transferred to the recipient (the last creditor). This failure leads to funds being locked in the contract, as the code does not handle such cases by default. Furthermore, the contract's reset process is triggered, but without paying the recipient, leaving the jackpot amount (profitFromCrash) stuck in the contract. Additionally, the send function does not throw an exception upon failure and only returns false. Since this false value is not checked in the contract, the contract continues to execute as if the transfer succeeded, enabling the attacker to exploit this flaw.

## References
[1] Ziyuan Li, Wangshu Guo, Quan Xu, Yingjie Xu, Huimei Wang, and Ming Xian. Research on blockchain smart contracts vulnerability and a code
audit tool based on matching rules. In Proceedings of the 2020 International Conference on Cyberspace Innovation of Advanced Technologies, pages
484–489, 2020.
[2] https://hackernoon.com/smart-contract-attacks-part-2-ponzi-games-gone-wrong-d5a8b1a98dd8
