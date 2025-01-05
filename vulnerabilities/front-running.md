# Front running
Front-running, also known as Transaction-Ordering Dependency (TOD) [1], is a vulnerability stemming from the inherent concurrency issues within blockchain systems. In blockchains, the execution order of transactions depends on miners' decisions, typically influenced by transaction rewards. This opens the door for exploitation, where a malicious externally owned account (EOA) can manipulate the system by offering a higher gas price to prioritize their transaction. Additionally, a malicious miner can bypass gas price considerations entirely, prioritizing their own transactions to gain an unfair advantage. This behavior undermines the fairness and integrity of decentralized systems.

## Toye Example
```Solidity
pragma solidity ^0.8.0;

contract VulnerableDEX {
    uint256 public tokenPrice = 1 ether; // Initial price of 1 token
    uint256 public constant PRICE_INCREMENT = 0.1 ether; // Price increases by 0.1 ETH after each purchase
    mapping(address => uint256) public balances;

    event TokensPurchased(address indexed buyer, uint256 amount, uint256 price);

    // Allows users to buy tokens
    function buyTokens(uint256 numTokens) public payable {
        uint256 totalPrice = numTokens * tokenPrice; // Calculate total price for the requested tokens
        require(msg.value >= totalPrice, "Insufficient ETH sent");

        balances[msg.sender] += numTokens;
        emit TokensPurchased(msg.sender, numTokens, tokenPrice);

        // 🔴 Vulnerability: The price is updated after each purchase. 
        // Attackers can exploit this by submitting a higher-gas transaction
        // to purchase tokens first, increasing the price for subsequent buyers.
        tokenPrice += PRICE_INCREMENT;
    }
}

```
## Real World Example
In the real world, numerous cases of front-running have led to significant financial crimes and heavy penalties. For instance, in 2009, 14 Wall Street firms faced fines amounting to nearly $70 million from the SEC (Securities and Exchange Commission). These examples highlight the real-world implications of such vulnerabilities and underscore the importance of addressing them effectively in blockchain systems [2].
The ERC-20 Double-Spend Attack, discovered in 2017 [3] during the initial implementation of ERC-20 tokens, targets the widely-used approve and transferFrom methods. This vulnerability poses a significant risk to all tokens adhering to the ERC-20 standard, as it can lead to unauthorized token transfers. Given the widespread adoption of ERC-20 on Ethereum, this systemic issue has had varied but substantial financial impacts globally. Below, we delve into the mechanics of this attack.
```Solidity
contract ERC20Token {
    mapping(address => mapping(address => uint256)) public allowance; // Stores allowances
    mapping(address => uint256) public balanceOf; // Tracks user balances

    function approve(address spender, uint256 value) public returns (bool success) {
        allowance[msg.sender][spender] = value; // 🛑 Vulnerable: Directly overwrites existing allowance
        emit Approval(msg.sender, spender, value);
        return true;
    }

    function transferFrom(address from, address to, uint256 value) public returns (bool success) {
        require(value <= allowance[from][msg.sender], "Insufficient allowance");
        require(value <= balanceOf[from], "Insufficient balance");

        allowance[from][msg.sender] -= value; // Reduces allowance after the transfer
        balanceOf[from] -= value;
        balanceOf[to] += value;

        emit Transfer(from, to, value);
        return true;
    }
}

```
- Initial Setup:
Alice allows Bob to spend a certain amount of her tokens by calling the approve function:
```Solidity
token.approve(bob, N);
```
- Changing Allowance:
Alice decides to update Bob's allowance to a new value and calls the approve function again:
```Solidity
token.approve(bob, M);
```
- Exploitation by Bob:
Bob observes Alice’s transaction before it is mined and quickly submits a transferFrom transaction to use the original allowance:
```Solidity
token.transferFrom(alice, bob, N); // Bob spends the old allowance
```
- Second Transfer:
After Alice’s new approve transaction is processed, Bob exploits the updated allowance to transfer additional tokens:
```Solidity
token.transferFrom(alice, bob, M); // Bob spends the new allowance
```
Due to this vulnerability, Bob can transfer up to N+M tokens, which is more than Alice intended. This issue arises because the approve function overwrites the previous allowance without validating whether the previous allowance was used or not.

### Why This Code is Vulnerable
- Lack of Atomicity: The approve function does not validate or clear the previous allowance before updating it. This makes it possible for a spender to exploit the gap between transactions.
- Front-Running Opportunity: If Bob can observe Alice's transaction and send his own transaction with a higher gas price, he can execute his transaction before Alice’s, leading to a double-spend scenario.

## References
[1] Bo Jiang, Ye Liu, and Wing Kwong Chan. Contractfuzzer: Fuzzing smart contracts for vulnerability detection. In Proceedings of the 33rd ACM/IEEE
International Conference on Automated Software Engineering, pages 259–269, 2018.

[2] https://www.sofi.com/learn/content/front-running/

[3] https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit?pli=1&tab=t.0
