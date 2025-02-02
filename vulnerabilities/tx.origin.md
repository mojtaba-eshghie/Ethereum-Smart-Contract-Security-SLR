# Tx.origin
The vulnerability related to tx.origin arises because it identifies the address of the user who originated the transaction, rather than the immediate caller of a function. This is problematic in identity verification because tx.origin includes the initial sender, even if multiple contracts are involved in the transaction. This can lead to attacks where a malicious contract tricks the vulnerable contract into assuming the attacker is the legitimate user, enabling unauthorized actions like withdrawing Ether or transferring assets [1].
## Toy Example
```Solidity
contract Attacker {
    Victim public victimContract;

    constructor(address _victimContractAddress) {
        victimContract = Victim(_victimContractAddress);
    }

    function attack() public {
        victimContract.withdrawAll(); // 🔴 Exploits the `tx.origin` vulnerability in the Victim contract to withdraw Ether.
    }

    receive() external payable {} // Contract can receive Ether
}
```
## Real World Example
On July 27, 2021, a critical vulnerability in the THORChain Router contract led to a devastating exploit, resulting in the loss of $8,000,000. The incident exposed a fundamental design flaw in the contract's authorization mechanism, showcasing the severe risks associated with improper implementation of ownership checks.
The contract for THORChain Hack #2 [2] is unavailable, but the code provided [3] effectively demonstrates the same vulnerability and serves as a suitable example for analysis. This function uses tx.origin to identify the sender, which is fundamentally flawed. The issue lies in the fact that tx.origin returns the original sender's address for the transaction, even if the transaction was initiated through an intermediary contract. This allows attackers to deceive users into calling a malicious contract, which in turn executes this function. In such cases, tx.origin still returns the address of the original user, enabling the attacker to transfer the user's assets without their direct consent.
The vulnerability lies in the following block of code, where the _transfer function is called, relying on tx.origin for sender verification:
```Solidity
function transferFunds(address recipient, uint256 amount) public {
    _transfer(tx.origin, recipient, amount);  
}

```
### How the Exploit Works
The vulnerability arises because tx.origin traces the original sender of a transaction, even if the transaction passes through multiple intermediary contracts. This behavior enables attackers to craft malicious contracts that exploit users interacting with them. Here's how the attack unfolds:
- Crafting the Malicious Contract:
The attacker creates a malicious contract that interacts with the vulnerable function in the THORChain Router contract.
- Deceiving the User:
The attacker tricks the legitimate user into calling a function on the malicious contract, believing it to be safe.
- Passing tx.origin:
When the malicious contract executes the _transfer function in the THORChain Router, tx.origin still reflects the original sender (the legitimate user) instead of the intermediary malicious contract.
- Unauthorized Fund Transfer:
The _transfer function erroneously authorizes the transfer based on tx.origin, allowing the attacker to redirect funds from the legitimate user to their account without the user's direct consent.
- Funds Redirected:
The attacker ensures the funds are sent to their controlled address (recipient), effectively stealing from the legitimate user.


## References
[1] iaming Ye, Mingliang Ma, Yun Lin, Lei Ma, Yinxing Xue, and Jianjun Zhao. Vulpedia: Detecting vulnerable ethereum smart contracts via abstracted vulnerability signatures. Journal of Systems and Software, 192:111410, 2022.

[2] https://rekt.news/thorchain-rekt2/

[3] ETH_RUNE Contract on Etherscan](https://etherscan.io/address/0x3624525075b88b24ecc29ce226b0cec1ffcb6976#code)



