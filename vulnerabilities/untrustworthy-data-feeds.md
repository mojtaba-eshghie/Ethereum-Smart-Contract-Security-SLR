# Untrustworthy Data Feeds
The vulnerability known as oracle/price manipulation is related to the use of oracles as it stems from the lack of trust in external data sources. Attackers can provide inaccurate information through oracles to disrupt SC operations. In DeFi protocols, such attacks often involve flash loans, which let users borrow assets without collateral, as long as they are repaid within the same transaction block. Attackers exploit this by temporarily manipulating token prices on platforms such as Uniswap, making profits before prices can be corrected.

## Toy Example
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract VulnerableDeFi {
    uint256 public constant BASE_PRICE = 1e18; // Base token price
    uint256 public totalSupply = 1e6 * 1e18;  // Total supply of the reward token

    mapping(address => uint256) public balances;
    address public rewardToken;

    constructor(address _rewardToken) {
        rewardToken = _rewardToken;
    }

    function deposit(uint256 amount) external {
        balances[msg.sender] += amount;
    }

    function getReward() external {
        uint256 rewardAmount = calculateReward(msg.sender);
        require(rewardAmount > 0, "No rewards available");

        // Transfer reward to the user
        ERC20(rewardToken).transfer(msg.sender, rewardAmount);
    }

    function calculateReward(address user) internal view returns (uint256) {
        uint256 userBalance = balances[user];
        uint256 poolBalance = getPoolReserves();  
        return userBalance * tokenPrice(poolBalance);  // Uses manipulated price
    }

    function tokenPrice(uint256 poolReserves) internal view returns (uint256) {
        //  Price is calculated based on pool reserves only!
        // If an attacker manipulates `poolReserves` using a flash loan,
        // they can artificially inflate the price and claim excessive rewards.
        return BASE_PRICE * (totalSupply / poolReserves);
    }

    function getPoolReserves() public view returns (uint256) {
        return ERC20(rewardToken).balanceOf(address(this));
    }
}

interface ERC20 {
    function transfer(address to, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

```

## Real World Example
On May 19, 2021, the Pancake Bunny protocol on the Binance Smart Chain (BSC) suffered a flash loan attack, resulting in a $45 million financial loss​.
The attacker first borrowed a large amount of tokens via a flash loan and swapped them for another token within the protocol. This action drastically reduced the reserves of the swapped token in the liquidity pool, leading to an artificial increase in its price. The attacker then exploited the protocol’s reward mechanism, which calculated reward distributions based on the manipulated token price. By triggering the getReward function at the inflated price, the attacker was able to obtain an abnormally high number of reward tokens. Finally, after selling these tokens on the market, the attacker repaid the flash loan and profited significantly from the attack.
```Solidity
function tokenPrice(uint256 poolReserves) internal view returns (uint256) {
    return BASE_PRICE * (TOTAL_SUPPLY / poolReserves);  
}

```
Since poolReserves can be manipulated using flash loans, an attacker can temporarily reduce its value, artificially inflating the token price. This inflated price is then used in the calculateReward() function, allowing the attacker to claim excessive rewards before the system corrects itself. The absence of a price oracle or TWAP (Time-Weighted Average Price) validation makes this exploit possible.

The root cause of this vulnerability was the protocol’s direct reliance on the instantaneous price of tokens in its reward calculation formula. Since flash loans allow attackers to manipulate token prices within a single transaction, they can exploit these price inconsistencies without any initial capital investment.
## References

https://bscscan.com/address/0xc9849e6fdb743d08faee3e34dd2d1bc69ea11a51#code

Mojtaba Eshghie, Mikael Jafari, and Cyrille Artho. From Creation to Exploitation: The Oracle Lifecycle. In 2024 IEEE International Conference on
Software Analysis, Evolution and Reengineering - Companion (SANER-C), pages 23–34, March 2024

Kong, Q., Chen, J., Wang, Y., Jiang, Z., & Zheng, Z. (2023, July). Defitainter: Detecting price manipulation vulnerabilities in defi protocols. In Proceedings of the 32nd ACM SIGSOFT International Symposium on Software Testing and Analysis (pp. 1144-1156).
