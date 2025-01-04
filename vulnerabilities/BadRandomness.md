# Bad Randomness
This Vulnerability refers to vulnerabilities in smart contracts that rely on pseudo-random number generation using blockchain variables such as block.timestamp, block.difficulty, block.number, or blockhash. These variables are not truly random and can be manipulated, especially by miners, within a limited range. Such manipulation enables malicious actors to predict or alter outcomes in systems like lotteries, gambling, or other applications requiring randomness. For example, a lottery contract that uses block.timestamp as a seed for generating random numbers is susceptible to miners tweaking the timestamp to influence the results in their favor. This vulnerability compromises fairness, leads to financial losses, and undermines user trust in the affected applications [1].

## Toye Example 
```Solidity
pragma solidity ^0.8.0;

contract InsecureRandom {
    uint256 public randomResult;

    /**
     * @dev Generates a "random" number using block.timestamp, block.difficulty, and msg.sender.
     * ⚠ WARNING: This method is insecure and prone to manipulation.
     * - `block.timestamp`: Miners can manipulate the timestamp within a reasonable range.
     * - `block.difficulty`: Predictable, especially in stable difficulty environments.
     * - `msg.sender`: Controlled by the user calling the function.
     */
    function getRandomNumber() public {
        randomResult = uint256(keccak256(abi.encodePacked(block.timestamp, block.difficulty, msg.sender)));
        // 🔴 Vulnerability: Using block.timestamp and block.difficulty makes the randomness predictable and manipulable.
    }
}
```

## Real World Example
In October 2017, the SmartBillions contract, a lottery-like smart contract, suffered from a critical vulnerability in its randomness generation mechanism. The vulnerability allowed miners or attackers to manipulate the random number generation process, compromising the fairness of the lottery and leading to significant financial losses of approximately 400 ETH (~$120,000 at the time). The issue was rooted in the predictability of inputs used for generating randomness. While the analyzed contract may not be the exact one exploited, it shares the same flawed design, making it a valuable case study for understanding predictable randomness vulnerabilities in smart contracts.

In the following code block, the vulnerability lies specifically in the line where randomness is generated using keccak256 with msg.sender and block.number as inputs [2]:
```Solidity
function play() payable public returns (uint) {
    return playSystem(uint(keccak256(msg.sender, block.number)), address(0));
}

```
The vulnerability allows attackers to manipulate the outcome of the lottery. Here's how the exploit works in practice:
- Input Control: The attacker has complete control over the msg.sender parameter since it represents the address of the transaction sender. By using multiple addresses or smart contracts, they can influence the randomness calculation. Block Number Manipulation: Miners can manipulate the block.number parameter by controlling when a transaction is included in the block. This allows them to predict or influence the random number generated.
- Attackers can precompute the result of the keccak256 function using different combinations of msg.sender and block.number to identify favorable outcomes. Once an attacker determines favorable inputs, they execute the function at the desired block number with the manipulated sender address, guaranteeing a win in the lottery. This exploit undermines the fairness of the lottery system by allowing attackers to game the randomness mechanism. The predictable nature of msg.sender and block.number makes this approach ineffective for any application requiring secure randomness.
### How the Exploit Works:
- Attack Setup: The attacker repeatedly sends transactions with different addresses or delays their inclusion until the desired block is mined.
- Outcome Control: By controlling msg.sender and block.number, the attacker can precompute the result of keccak256 and choose parameters that produce a favorable outcome.
- Exploitation: Once the attacker finds favorable inputs, they execute the function, effectively rigging the lottery.

## References
[1] Bo Jiang, Ye Liu, and Wing Kwong Chan. Contractfuzzer: Fuzzing smart contracts for vulnerability detection. In Proceedings of the 33rd ACM/IEEE International Conference on Automated Software Engineering, pages 259–269, 2018.

[2] https://github.com/SmartBillions/SmartBillions/blob/master/SmartBillions.sol


