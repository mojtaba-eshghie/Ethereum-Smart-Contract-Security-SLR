# Tx.origin
The vulnerability related to tx.origin arises because it identifies the address of the user who originated the transaction, rather than the immediate caller of a function. This is problematic in identity verification because tx.origin includes the initial sender, even if multiple contracts are involved in the transaction. This can lead to attacks where a malicious contract tricks the vulnerable contract into assuming the attacker is the legitimate user, enabling unauthorized actions like withdrawing Ether or transferring assets [1].
## Toye Example
The Victim contract contains a vulnerability due to its use of tx.origin for authorization in the withdrawAll function. The contract checks if the transaction's origin matches the owner before allowing the withdrawal of all funds. However, using tx.origin can be exploited by an attacker through a malicious contract, allowing unauthorized access to funds.

```Solidity
contract Victim {
    address public owner;
    constructor() {
        owner = msg.sender;
    }
    function withdrawAll() public {
        require(tx.origin == owner, "Not authorized");
        payable(msg.sender).transfer(address(this).balance);
    }
    receive() external payable {}   
}
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

The vulnerability arises because tx.origin traces the original sender of a transaction, even if it passes through multiple intermediary contracts, allowing attackers to exploit this behavior. In the attack, the attacker crafts a malicious contract that interacts with the vulnerable function in the THORChain Router contract. They deceive the legitimate user into interacting with the malicious contract, believing it to be safe. As the malicious contract executes the _transfer function in the THORChain Router, tx.origin still reflects the original sender (the legitimate user) instead of the intermediary malicious contract. This results in an unauthorized fund transfer, as the _transfer function erroneously authorizes the transfer based on tx.origin, allowing the attacker to redirect funds to their own address, effectively stealing from the legitimate user without their direct consent.


## References
[1] iaming Ye, Mingliang Ma, Yun Lin, Lei Ma, Yinxing Xue, and Jianjun Zhao. Vulpedia: Detecting vulnerable ethereum smart contracts via abstracted vulnerability signatures. Journal of Systems and Software, 192:111410, 2022.

[2] https://rekt.news/thorchain-rekt2/

[3] ETH_RUNE Contract on Etherscan](https://etherscan.io/address/0x3624525075b88b24ecc29ce226b0cec1ffcb6976#code)



