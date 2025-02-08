# State Reverting Vulnerability
This type of vulnerability occurs when the contract’s state variables are exposed to external calls through the state-reverting mechanism. This exposure allows attackers to intentionally manipulate critical states, potentially exploiting the contract~\cite{liao2023smartstate}. An example application where this vulnerability can be exploited is dApps with probabilistic reward mechanisms by allowing an attacker to selectively accept favorable outcomes. An adversary interacts with the contract through an auxiliary contract, recording their balance before invoking a reward function. If an unfavorable outcome occurs, they trigger a condition that forces the contract to revert, effectively undoing the transaction. By repeating this process, the attacker guarantees receiving only the high-value reward, undermining fairness and causing financial losses. Mitigation strategies include committing outcomes before exposure, using verifiable randomness, and preventing repeated manipulation.

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

## References
