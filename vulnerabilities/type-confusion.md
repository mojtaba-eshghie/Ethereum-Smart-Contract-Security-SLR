# Type Confusion
Type confusion happens when the runtime confuses the type of an object with another type leading to its incorrect utilization. For instance, when a variable is allocated with one type and later accessed with an incompatible type. This can also lead to logical errors or out-of-bounds memory access for storage variables. A specific subcategory of this vulenrability is also known as storage collision. This specific subcategory happens when two contracts accessing the same storage space have a different understanding of the storage layout. This vulnerability is often is observed when a proxy upgradability design pattern is used. 
The proxy upgradability design pattern separates a contract’s state from its logic, allowing the system’s functionality to evolve without losing stored data. In this design, a proxy contract holds the persistent state and delegates calls to an external implementation contract using the low-level  delegatecall opcode [1].
## Toy Example
```Solidity
pragma solidity ^0.8.0;
contract Proxy {
 uint public visits; // Storage slot [0x0]
 address public LOGIC; // Storage slot [ERC-1967]
 constructor(address logicAddress) {
  LOGIC = logicAddress;
  (bool success, ) = LOGIC.delegatecall(abi.encodeWithSignature("initialize()"));
  require(success, "Initialization failed");
 }
 fallback() external payable {
  (bool success, ) = LOGIC.delegatecall(msg.data);
  require(success, "Delegatecall failed");
 }
}
contract Logic {
 bool public initialized; // Storage slot [0x0]
 address public admin; // Storage slot [0x0]
 uint[] public artworkIDs; // Storage slot [0x1]
 mapping(address => uint) public artworkHolders; // Storage slot [0x2]
 function initialize() external {
  require(!initialized, "Already initialized");
  initialized = true;
  admin = msg.sender; // Overwrites Proxy's storage slot [0x0]
 }
 function withdraw() external {
  require(msg.sender == admin, "Not admin");
  payable(admin).transfer(address(this).balance);
 }
}
```

## Real World Example
On July 23, 2022, Audius experienced a security breach where an attacker exploited a vulnerability in the contract initialization code, allowing them to repeatedly invoke the initialize functions. This exploit enabled the malicious transfer of 18 million $AUDIO tokens from the community treasury to the attacker's wallet and unauthorized modifications to the voting system's staked $AUDIO amounts. The vulnerability stemmed from a storage collision between the proxyAdmin address and OpenZeppelin's Initializable contract's state variables, causing the initializer modifier to always succeed. Despite prior audits by OpenZeppelin and Kudelski, this issue was not identified.
The example provided in the toy example section is actually a simulation of the real code, accurately demonstrating what happened in the actual contract.
This smart contract setup is vulnerable to a storage collision due to the improper alignment of storage slots between the Proxy and Logic contracts when using delegatecall Specifically both contracts store critical state variables in slot 0x0 but with different interpretations the Proxy contract uses it for visits while the Logic contract stores initialized and admin in the same slot When initialize is called via delegatecall it writes true 1 to initialized and sets admin to msg sender unknowingly overwriting the visits variable in the Proxy contract An attacker can exploit this by re invoking initialize since the overwritten storage allows it to be called again setting their own address as admin Once this is done they can execute withdraw to drain the contracts funds.
The example provided in the toy example section is actually a simulation of the real code, accurately demonstrating what happened in the actual contract.

## References
[1] Yao, S., Ni, H., Myers, A. C., & Cecchetti, E. (2024). SCIF: A Language for Compositional Smart Contract Security. arXiv preprint arXiv:2407.01204.

[2] [https://github.com/thorium-dev-group/dexible-contracts](https://blog.audius.co/article/audius-governance-takeover-post-mortem-7-23-22?utm_source=chatgpt.com)


