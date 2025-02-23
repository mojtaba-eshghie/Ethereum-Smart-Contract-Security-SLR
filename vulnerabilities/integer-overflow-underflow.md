## Description
This vulnerability was first identified during the attack on BEC tokens [1]. It occurs when the result of a calculation exceeds the maximum or minimum limit of the variable's data type, making it impossible to accurately represent the value. This issue is particularly prevalent in smaller data types. For instance, when a balance reaches the upper limit of  2^256 ,  it resets to zero. Furthermore, division operations on integers in Solidity can introduce rounding errors, which may result in unexpected discrepancies in calculations that require high precision [2]. While these issues may not directly constitute an integer overflow or underflow vulnerability, they can lead to logical flaws in the business logic of smart contracts.
## Toy Example


```Solidity
// This contract demonstrates integer overflow and underflow vulnerabilities.
// Older Solidity versions (<0.8.0) did not have built-in checks for these issues.

contract IntegerOverflowExample {
    mapping(address => uint256) public balances;
    uint256 public constant MAX_UINT = type(uint256).max;
    function deposit(uint256 amount) public {
        require(amount > 0, "Amount must be greater than zero");
        balances[msg.sender] += amount; // Possible overflow if not checked
    }
    function withdraw(uint256 amount) public {
        require(amount > 0, "Amount must be greater than zero");
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount; // Underflow risk if unchecked
    }
    function forceOverflow() public {
        balances[msg.sender] = MAX_UINT; // Assign max value
        balances[msg.sender] += 1; // This would have caused overflow in older Solidity versions (<0.8.0)
    }
}
```


## Real-World Example
In  July 2024 , a security vulnerability was exploited, leading to a loss of approximately  7K USD . The attack targeted the Lw contract, potentially leveraging an  integer overflow vulnerability . One of the problematic lines of code in this attack attempted to transfer an excessively large amount [3]:

```Solidity
Lw.transferFrom(address(Lw), address(this), 1_000_000_000_000_000_000_000_000_000_000_000);
```
If the contract lacks overflow protection, this value could exceed the storage capacity of uint256. In older versions of Solidity (prior to 0.8.0) or if SafeMath is not used, this operation could cause an integer overflow, where the value wraps around to a much smaller number, effectively bypassing intended restrictions.
nother critical issue was found in a loop that executed up to 9999 times, processing token transactions:
```Solidity
while (i < 9999) {
    swap_token_to_token(address(Lw), address(BUSDT), 800_000_000 ether);
    i++;
}

```
This code presents two major risks. First, if the variable i overflows, it could result in an unexpected infinite loop, causing unintended execution and potential denial-of-service (DoS) scenarios. Without proper checks, the loop may continue indefinitely, leading to excessive gas consumption and contract failure.

Second, the transaction amount of 800_000_000 ether passed to the swap_token_to_token function could pose a significant risk if not properly handled in the Lw or BUSDT contracts. If these contracts lack appropriate overflow protection, the large value might cause calculation errors or allow unintended balance manipulations, potentially enabling malicious actors to exploit the system for financial gain.

## References 
[1] Huashan Chen, Marcus Pendleton, Laurent Njilla, and Shouhuai Xu. A survey on ethereum systems security: Vulnerabilities, attacks, and defenses.
ACM Computing Surveys (CSUR), 53(3):1–43, 2020.

[2] Christof Ferreira Torres, Julian Schütte, and Radu State. Osiris: Hunting for integer bugs in ethereum smart contracts. In Proceedings of the 34th
annual computer security applications conference, pages 664–676, 2018.

[3] https://github.com/SunWeb3Sec/DeFiHackLabs
