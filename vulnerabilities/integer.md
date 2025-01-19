## Integer overflow/underflow
This vulnerability was first identified during the attack on BEC tokens [1]. It occurs when the result of a calculation exceeds the maximum or minimum limit of the variable's data type, making it impossible to accurately represent the value. This issue is particularly prevalent in smaller data types. For instance, when a balance reaches the upper limit of  2^256 ,  it resets to zero. Furthermore, division operations on integers in Solidity can introduce rounding errors, which may result in unexpected discrepancies in calculations that require high precision [2]. While these issues may not directly constitute an integer overflow or underflow vulnerability, they can lead to logical flaws in the business logic of smart contracts.
## Toy Example
```Solidity
pragma solidity ^0.7.0;

contract OverflowExample {
    uint8 public maxValue = 255; // Maximum value for uint8 (2^8 - 1)

    function add(uint8 _value) public returns (uint8) {
        uint8 result = maxValue + _value; // ⚠️ This line may cause an Overflow
        return result;
    }
}
```
```Solidity
pragma solidity ^0.7.0;

contract UnderflowExample {
    uint8 public minValue = 0; // Minimum value for uint8

    function subtract(uint8 _value) public returns (uint8) {
        uint8 result = minValue - _value; // ⚠️ This line may cause an Underflow
        return result;~
    }
}
```


## Real-World Example
On November 11, 2024, the DeltaPrime smart contract suffered a reentrancy attack, resulting in a loss of approximately $4.75M USD. The attacker exploited a vulnerability in the contract's reward claiming mechanism, which lacked proper checks to prevent reentrancy during the execution of flash loans and reward claims. By leveraging a fake pair contract, the attacker manipulated the smart loan's behavior to repeatedly claim rewards and convert collateral into WETH (Wrapped Ether) without proper validation. This exploit highlights the critical need for implementing reentrancy guards and thorough validation mechanisms in sensitive contract functions. In the following sections, we will highlight the specific vulnerable points in this contract.

- Reentrancy via call to attackContract
The claim function in the FakePairContract calls attackContract using call. This low-level function provides no checks or restrictions, enabling the attacker to execute arbitrary code. In this case, the attacker leverages the convertETH function from the DeltaPrimeExp contract during the reward claim process. Since the claim function does not employ a reentrancy guard (e.g., nonReentrant), the attacker can reenter the function multiple times within the same transaction. This allows them to repeatedly manipulate the state of the contract and claim rewards in an unintended manner.
```Solidity
function claim(address user, uint256[] calldata ids) external {
    // : Allows reentrancy by calling the exploit's convertETH function.
    attackContract.call(abi.encodeWithSelector(DeltaPrimeExp.convertETH.selector, ""));
}

```
- Chaining Reentrant Calls
The attacker utilizes reentrant calls to manipulate the collateral conversion process repeatedly. By calling the convertETH function from within the claim function, they withdraw WETH and convert it into ETH. This process is repeated, draining the contract of its assets before the state updates or checks can finalize. The vulnerability lies in the lack of a mechanism to ensure that the claim function can only be executed once per transaction.

- Flash Loan Exploitation
The attacker initiates the exploit with a large flash loan using the flashLoan function from Balancer. This provides the liquidity required to begin the exploit and manipulate the reward and collateral mechanisms. After the exploit completes, the attacker repays the flash loan, extracting the stolen assets as profit. For instance:

```Solidity
Balancer.flashLoan(address(this), tokens, amounts, userData);
```
 
- Unchecked Collateral Conversion
The convertETH function plays a critical role in the exploit. By calling it repeatedly, the attacker converts the collateral into WETH, which is then claimed as a reward. The collateral is continuously manipulated without proper validation of balances or limits:

```Solidity
address(SmartLoan).call(wrapNativeTokenData);
```

- Draining WETH Balance
After completing the reentrant calls, the attacker transfers the WETH balance to the Balancer flash loan pool to finalize the flash loan repayment:

```Solidity
WETH.transfer(address(Balancer), flashLoanAmount);
```
This step ensures that the attacker extracts the profit without leaving any traceable balance within the vulnerable contract.

## References 
[1] Huashan Chen, Marcus Pendleton, Laurent Njilla, and Shouhuai Xu. A survey on ethereum systems security: Vulnerabilities, attacks, and defenses.
ACM Computing Surveys (CSUR), 53(3):1–43, 2020.

[2] Christof Ferreira Torres, Julian Schütte, and Radu State. Osiris: Hunting for integer bugs in ethereum smart contracts. In Proceedings of the 34th
annual computer security applications conference, pages 664–676, 2018.

[3] https://github.com/SunWeb3Sec/DeFiHackLabs
