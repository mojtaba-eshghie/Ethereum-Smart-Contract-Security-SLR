# State Reverting Vulnerability
This type of vulnerability occurs when the contract’s state variables are exposed to external calls through the state-reverting mechanism. This exposure allows attackers to intentionally manipulate critical states, potentially exploiting the contract. An example application where this vulnerability can be exploited is dApps with probabilistic reward mechanisms by allowing an attacker to selectively accept favorable outcomes. An adversary interacts with the contract through an auxiliary contract, recording their balance before invoking a reward function. If an unfavorable outcome occurs, they trigger a condition that forces the contract to revert, effectively undoing the transaction [1]. By repeating this process, the attacker guarantees receiving only the high-value reward, undermining fairness and causing financial losses. Mitigation strategies include committing outcomes before exposure, using verifiable randomness, and preventing repeated manipulation.

## Toy Example
```Solidity
contract TokenGame {
    mapping(address => uint256) public SheepToken;
    mapping(address => uint256) public WolfToken;

    function MintToken(address account) public {
        uint256 seed = (random() >> 245) % 10;

        if (seed != 0) {
            SheepToken[account]++;
        } else {
            // The attacker exploits this condition
            WolfToken[account]++;
        }
    }
}

contract Attacker {
    function onlyWolf(TokenGame target, uint256 tokenId) public {
        uint256 Before = target.WolfToken(address(this));
        target.MintToken(tokenId);
        uint256 After = target.WolfToken(address(this));

        // If the received token is undesirable, revert the transaction
        require(After > Before);
    }
}

```

## Real World Example

On April 23, 2022, the Akutars project, an Ethereum-based NFT collection, suffered a logical error in its smart contract. This bug resulted in the permanent locking of 11,539.5 ETH (approximately $34 million at the time) in the contract, leaving both developers and users unable to recover the funds.
The issue occurred in the processRefunds() function, which was responsible for processing refunds to auction participants. A logical flaw prevented the variable refundsProcessed from being set, causing the contract to lock the funds indefinitely [2].


```Solidity
function processRefunds(uint256 _bidIndex) external nonReentrant {
    require(!refundsProcessed, "Refunds already processed");
    require(_bidIndex < totalBids, "Invalid bid index");

    for (uint256 i = _bidIndex; i < totalBids; i++) {
        address bidder = bidders[i];
        uint256 bidAmount = bids[bidder];

        if (bidAmount > 0) {
            bids[bidder] = 0;
            (bool success, ) = bidder.call{value: bidAmount}("");
            require(success, "Refund failed");
        }

        refundProgress++;
    }

    if (refundProgress >= totalBids) {
        refundsProcessed = true;
    }
}

Based on the code above, the conditional statement if (refundProgress >= totalBids) would only be satisfied if all refunds had been processed. However, due to certain conditions, refundProgress was not correctly incremented, causing the condition to never be met. As a result, refundsProcessed was never assigned a value, and the processRefunds() function consistently triggered revert(). Since this condition was always false, any attempt to execute the function resulted in a reverted transaction, preventing the withdrawal of funds. Consequently, no one, including the development team, was able to retrieve the locked funds, leading to their permanent loss.
## References
[1] Liao, Zeqin, et al. "Smartstate: Detecting state-reverting vulnerabilities in smart contracts via fine-grained state-dependency analysis." Proceedings of the 32nd ACM SIGSOFT International Symposium on Software Testing and Analysis. 2023.
[2] https://etherscan.io/address/0xf42c318dbfbaab0eee040279c6a2588fa01a961d#code
