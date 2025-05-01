#  Compiler Version Not Fixed
This vulnerability occurs when a contract uses an outdated compiler, exposing it to bugs and security risks. If pragma solidity  0.4.0; is used, the contract can be compiled with any 0.4.x version, including vulnerable ones.  
For example, in versions before 0.4.22, a function with the same name as the contract (e.g., function Vulnerable() public) was treated as a regular function instead of a constructor, making it callable by attackers. However, if pragma solidity 0.4.25; is used instead, the contract is only compilable with version 0.4.25, where constructors must be explicitly defined using the constructor keyword (e.g., constructor() public), preventing this vulnerability by restricting compilation to a safe version.
## Toy Example
```Solidity
//  Vulnerable: Compiler version not fixed
pragma solidity ^0.8.0;  // Allows any version >=0.8.0 <0.9.0

contract VulnerableContract {
    uint public value;

    function setValue(uint _value) public {
        require(_value > 0, "Value must be positive");
        value = _value;
    }

    function getValue() public view returns (uint) {
        return value;
    }
}

```

## Real World Example
The Rubixi smart contract, originally deployed in 2017, suffered from a constructor naming vulnerability, leading to the theft of approximately 100 ETH (worth around $150,000 at the time). The issue arose when the contract was renamed from DynamicPyramid to Rubixi, but the constructor function retained its original name (DynamicPyramid). Since Solidity versions prior to 0.4.22 required the constructor name to match the contract name, this function was treated as a regular public function instead of a constructor. As a result, anyone could call it and set themselves as the contract owner, ultimately withdrawing all funds.

```Solidity
contract Rubixi {
    address private creator;

    function DynamicPyramid() {
        creator = msg.sender;
    }
}

```
The DynamicPyramid() function was intended to be the constructor, but after the contract was renamed to Rubixi, Solidity no longer recognized it as such. Since Solidity versions prior to 0.4.22 treated constructors as functions with the same name as the contract, this function remained callable after deployment. This allowed an attacker to invoke DynamicPyramid(), overwrite the creator address, and gain full control over the contract. As a result, the attacker could withdraw all stored ETH using functions restricted to onlyowner, leading to significant financial loss.

## REferences

[1] https://github.com/crytic/not-so-smart-contracts/blob/master/wrong_constructor_name/Rubixi_source_code/Rubixi.sol

[2] Demir, M., Alalfi, M., Turetken, O., & Ferworn, A. (2019, July). Security smells in smart contracts. In 2019 IEEE 19th International Conference on Software Quality, Reliability and Security Companion (QRS-C) (pp. 442-449). IEEE.
