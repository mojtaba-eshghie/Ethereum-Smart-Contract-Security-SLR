# Ponzi
Ponzi contracts exploit design flaws, leading to vulnerabilities due to poor transparency, mismanagement, false promises of high returns, and legal non-compliance. These schemes often create an illusion of profitability by relying on the continuous influx of new investors, and the promised high returns are paid from the investments of newer participants rather than actual profits. This deceptive mechanism leads to financial loss for later investors and is marked by a lack of accountability and sustainability, ultimately resulting in the collapse of the system once the flow of new investments slows down or stops.

## Toy Example
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimplePonzi {
    uint256 public totalInvested;
    mapping(address => uint256) public balances;

    // Investment in the Ponzi scheme
    function invest() public payable {
        require(msg.value > 0, "Send some ETH to invest!"); // Ensure investment is greater than 0
        balances[msg.sender] += msg.value; //🔴 Add the investment to the sender's balance
        totalInvested += msg.value; //🔴 Update the total invested amount
    }

    // Claiming reward (payment from new investors)
    function claimReward() public {
        uint256 reward = balances[msg.sender] / 10; // 10% reward (the reward paid from new investors)
        require(reward > 0, "No reward available"); // Ensure the user has reward to claim

        // Pay reward to the investor
        payable(msg.sender).transfer(reward); //🔴 Transfer the reward to the sender
        balances[msg.sender] = 0; // Reset investment after reward is claimed
    }

    // View the balance of the investor
    function getBalance() public view returns (uint256) {
        return balances[msg.sender]; // View the balance of the caller
    }
}
```
## Real World Example
The Daily12 contract [1] had a lifecycle of 10 days, during which it rapidly accumulated 146.59 ETH in just 3 days but lost all its balance in the subsequent 7 days. A total of 171.04 ETH (around $219,782.98 at the time) was involved, reflecting substantial financial activity. The Ponzi scheme mechanism promised participants a daily 12% return on investments, but these returns were paid out from the investments of newer participants, creating an illusion of sustainability for approximately 8.33 days (100% divided by 12%). The contract relied on continuous new investments to pay earlier participants, with profits only being achievable for those who invested early, particularly before block height 6547454. Early investors who reinvested often faced losses due to the degenerating nature of the contract. The source code, available on etherscan.io, implemented the Ponzi logic using mappings to record investments and calculate payouts based on the promised returns. The contract below displays the code for this contract.
```Solidity
contract Daily12 {
    mapping (address => uint256) public invested;
    mapping (address => uint256) public atBlock;

    function () external payable {
        if (invested[msg.sender] != 0) {
            uint256 amount = invested[msg.sender] * 12 / 100 * (block.number - atBlock[msg.sender]) / 5900;
            msg.sender.transfer(amount);
        }
        atBlock[msg.sender] = block.number;
        invested[msg.sender] += msg.value;
    }
}
```
The Daily12 contract follows the logic of a Ponzi scheme. It records the investments of participants using the investments mapping, where each participant's address maps to their total invested amount. Additionally, the contract tracks the block number when participants retrieve their profits using the lastTransaction mapping.

When a participant calls the invest() function, the contract first retrieves the sender's last transaction block with lastTransaction[msg.sender]. Then, it calculates the time interval by subtracting the last transaction block from the current block number with block.number - lastBlock. Using this time interval, the contract computes 12% of the sender's investment scaled by the time interval: (investmentAmount * 12 / 100) * timeInterval.

After calculating the profit, the contract transfers the computed amount back to the sender with payable(msg.sender).transfer(profit). The contract then updates the sender's last transaction block with lastTransaction[msg.sender] = block.number and adds the new investment value to their total with investments[msg.sender] += investmentAmount.

This is how the Daily12 contract operates as a Ponzi scheme, distributing returns from new investments to earlier participants.

## References

[1] Zheng, Z., Chen, W., Zhong, Z., Chen, Z., & Lu, Y. (2023). Securing the ethereum from smart ponzi schemes: Identification using static features. ACM Transactions on Software Engineering and Methodology, 32(5), 1-28.


