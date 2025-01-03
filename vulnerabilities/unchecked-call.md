# Unchecked call return value
This vulnerability, referred to as "Unchecked External Call," "Mishandled Exception," or "Exception Disorder," arises from improper handling of asynchronous exceptions in Smart Contracts. If an exception occurs during an external call, the remaining contract operations might not execute as intended. Two primary variations of this issue are Unchecked Send and Gasless Send~\cite{zhang2023svscanner}. In Gasless Send, sending Ether using a gasless method can trigger an out-of-gas exception if the receiving contract's fallback function is limited to 2300 gas units, enabling malicious actors to unlawfully retain Ether. Unchecked Send involves low-level calls (e.g., send, call, delegatecall, callcode) where an exception in the callee contract returns false instead of propagating the exception, potentially causing unintended transactions [1].

## Toy Example
```Solidity
contract UncheckedReturn {
        // Function to send Ether to a target address
        function sendEther(address payable _target) public payable {
            // Attempt to send Ether without checking the return value
            _target.transfer(msg.value); // No check for success
            // If the transfer fails, it could lead to unexpected behavior
        }
    }
```
## Real World Example
In 2016, a smart contract named King of the Ether Throne (KotET)[2], which ultimately resulted in the permanent loss of all Ether within it (estimated to be a significant amount at the time), gained attention as a decentralized game where players could claim the throne by sending Ether to the contract. The game’s mechanics revolved around a simple but enticing concept: each new King paid a claim price to take the throne, and the dethroned King would receive compensation. As the number of Kings increased, so did the cost of claiming the throne. If no new King emerged within 14 days, the throne would reset, and the game would start anew. This game, inspired by the “Greater Fool’s Theory,” attracted participants eager to profit, but it harbored a critical vulnerability that led to its demise.
The vulnerability in the KotET contract, categorized as an Exception Disorder, had severe consequences. The issue centered around a single line of code: king.send(compensation);. The contract relied on the send() function to transfer Ether to the dethroned King. However, the send() function enforces a gas limit of 2300 gas, which is insufficient for contracts with fallback functions requiring more gas. If the dethroned King was a contract with a complex fallback function, the transfer operation would silently fail. This oversight caused a catastrophic failure in the game’s mechanics.
The highlighted flaw lay in the contract’s lack of error handling. When the send() function failed, the contract neither checked its return value nor implemented a mechanism to address the failure. As a result, the dethroned King could not receive the compensation, and no new King could take the throne. This led to a deadlock, permanently locking all Ether within the contract. Analyzing the contract’s code reveals the vulnerability clearly. In the fallback function:
 ```Solidity
if (msg.value < claimPrice) revert;
uint compensation = calculateCompensation();        
king.send(compensation);
king = msg.sender;
claimPrice = calculateNewPrice();
```
The use of king.send(compensation); was particularly problematic. By enforcing a 2300 gas limit, it assumed that all Kings would have simple fallback functions. However, if the current King’s address was a smart contract with a more complex fallback function, the transfer would fail. The failure was compounded by the absence of a mechanism to verify whether the send() call succeeded. This left the throne inaccessible and the game inoperable.
Additionally, the sweepCommission() function, meant for the owner to withdraw fees, also relied on owner.send(amount);. While not directly related to the throne-locking issue, it exhibited the same vulnerability, potentially leading to failed withdrawals if the owner’s address was a contract.

## References
[1] Hengyan Zhang, Weizhe Zhang, Yuming Feng, and Yang Liu. Svscanner: Detecting smart contract vulnerabilities via deep semantic extraction. Journal of Information Security and Applications, 75:103484, 2023

[2] https://medium.com/hackernoon/smart-contract-attacks-part-2-ponzi-games-gone-wrong-d5a8b1a98dd8


