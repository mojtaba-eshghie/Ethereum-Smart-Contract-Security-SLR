# Bad Randomness
This Vulnerability refers to vulnerabilities in smart contracts that rely on pseudo-random number generation using blockchain variables such as block.timestamp, block.difficulty, block.number, or blockhash. These variables are not truly random and can be manipulated, especially by miners, within a limited range. Such manipulation enables malicious actors to predict or alter outcomes in systems like lotteries, gambling, or other applications requiring randomness. For example, a lottery contract that uses block.timestamp as a seed for generating random numbers is susceptible to miners tweaking the timestamp to influence the results in their favor. This vulnerability compromises fairness, leads to financial losses, and undermines user trust in the affected applications [1].

## Toye Example 
```Solidity
pragma solidity ^0.8.0;
contract InsecureRandom {
    uint256 public randomResult;
    function getRandomNumber() public {
        randomResult = uint256(keccak256(abi.encodePacked(block.timestamp, block.difficulty, msg.sender)));
        // Using block.timestamp and block.difficulty makes the randomness predictable and manipulable.
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
 This vulnerability allows attackers to manipulate the lottery’s outcome by exploiting predictable inputs in the randomness mechanism. Since msg.sender represents the transaction sender’s address, an attacker can use multiple addresses or smart contracts to influence the random number generation. Additionally, miners can control the block.number parameter by choosing when to include a transaction in a block, further manipulating the outcome. By precomputing the keccak256 hash with different combinations of msg.sender and block.number, attackers can determine favorable inputs in advance. Once an optimal combination is found, they execute the transaction at the desired block, ensuring a guaranteed win.

This exploit undermines the fairness of the lottery by making the randomness predictable. Attackers can repeatedly submit transactions with different addresses or delay their inclusion in a block until they achieve a favorable result. Since both msg.sender and block.number are insufficient for secure randomness, any application relying on them for unpredictable outcomes remains vulnerable to manipulation.

## References
[1] Bo Jiang, Ye Liu, and Wing Kwong Chan. Contractfuzzer: Fuzzing smart contracts for vulnerability detection. In Proceedings of the 33rd ACM/IEEE International Conference on Automated Software Engineering, pages 259–269, 2018.

[2] https://github.com/SmartBillions/SmartBillions/blob/master/SmartBillions.sol


