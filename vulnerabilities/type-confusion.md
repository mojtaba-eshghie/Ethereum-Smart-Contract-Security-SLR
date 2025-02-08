# Type Confusion
Type confusion happens when the runtime confuses the type of an object with another type leading to its incorrect utilization. For instance, when a variable is allocated with one type and later accessed with an incompatible type. This can also lead to logical errors or out-of-bounds memory access for storage variables. A specific subcategory of this vulenrability is also known as \emph{storage collision}. This specific subcategory happens when two contracts accessing the same storage space have a different understanding of the storage layout. This vulnerability is often is observed when a proxy upgradability design pattern is used. 
The proxy upgradability design pattern separates a contract’s state from its logic, allowing the system’s functionality to evolve without losing stored data. In this design, a proxy contract holds the persistent state and delegates calls to an external implementation contract using the low-level \texttt{delegatecall} opcode. [1].
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
The Dexible attack on February 17, 2023, resulted in the theft of approximately $2 million USD due to a vulnerability in the delegatecall function, which allowed execution of user-specified data against an arbitrary target address (to) without validation. This function enabled attackers to execute malicious contract logic in Dexible’s storage context. The lack of validation allowed attackers to overwrite critical variables and misuse storage layouts, leading to unauthorized token transfers and loss of control over the contract.
The paper [1] mentions the Dexible contract as an example of a confused deputy attack (CDA), which it connects to the concept of type confusion. The specific vulnerability lies in the swap function, where an untrusted user could supply a malicious callback or contract address to the router parameter. The system lacked verification of the method or type of this router, resulting in an unsafe delegate call. This mismatch between the assumed type of the router parameter by the caller and the actual type of the provided malicious contract demonstrates a classic type confusion scenariocode highlighting the vulnerable block is as follows:
```Solidity
contract Dexible {
    function swap(uint amount, address tokenIn, address router, bytes memory routerData) external {
        if (IERC20(tokenIn).transferFrom(msg.sender, address(this), amount)) {
            IERC20(tokenIn).safeApprove(router, amount);
            // 🔴 Vulnerable Line: No type or method validation on router
            (bool succ, ) = router.call(routerData);
            assert(succ, "Failed to swap");
        } else {
            revert("Insufficient balance");
        }
    }
}
```
The vulnerability arises because the router parameter, which is an address, can point to any contract without validation of its type or the methods it supports. The (bool succ, ) = router.call(routerData) line dynamically calls the provided contract at router with arbitrary data, effectively allowing an attacker to exploit Dexible's authority and perform unauthorized actions, such as transferring funds.

This demonstrates type confusion because the expected type of router (a legitimate exchange contract) is mismatched with the type of the malicious contract provided by the attacker. The lack of type enforcement or method signature validation creates an opportunity for exploitation.

## References
[1] Yao, S., Ni, H., Myers, A. C., & Cecchetti, E. (2024). SCIF: A Language for Compositional Smart Contract Security. arXiv preprint arXiv:2407.01204.

[2] https://github.com/thorium-dev-group/dexible-contracts

[3] https://blockapex.io/dexible-hack-analysis/?utm_source=chatgpt.com


