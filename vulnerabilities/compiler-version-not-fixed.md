#  Compiler Version Not Fixed
This vulnerability occurs when a contract uses an outdated compiler, exposing it to bugs and security risks. If pragma solidity  0.4.0; is used, the contract can be compiled with any 0.4.x version, including vulnerable ones.  
For example, in versions before 0.4.22, a function with the same name as the contract (e.g., function Vulnerable() public) was treated as a regular function instead of a constructor, making it callable by attackers. However, if pragma solidity 0.4.25; is used instead, the contract is only compilable with version 0.4.25, where constructors must be explicitly defined using the constructor keyword (e.g., constructor() public), preventing this vulnerability by restricting compilation to a safe version.
## Toy Example


## Real World Example


## REferences
