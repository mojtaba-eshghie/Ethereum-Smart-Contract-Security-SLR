# Type Confusion
Vulnerability occurs when a programmer mistakenly converts an object to a different type or incorrectly utilizes it, like a pointer or variable is allocated with one type and later accessed with an incompatible type. This can lead to logical errors or out-of-bounds memory access [1].
## Toy Example
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Proxy contract
contract Proxy {
    address public implementation; // Address of the logic contract
    address public owner; // Owner of the proxy contract

    constructor(address _implementation) {
        implementation = _implementation;
        owner = msg.sender;
    }

    function updateImplementation(address _newImplementation) external {
        require(msg.sender == owner, "Not authorized");
        implementation = _newImplementation;
    }

    // Delegatecall forwards calls to the implementation contract
    fallback() external payable {
        address impl = implementation;
        require(impl != address(0), "Implementation not set");

        assembly {
            let ptr := mload(0x40)
            calldatacopy(ptr, 0, calldatasize())
            let result := delegatecall(gas(), impl, ptr, calldatasize(), 0, 0)
            let size := returndatasize()
            returndatacopy(ptr, 0, size)
            switch result
            case 0 { revert(ptr, size) }
            default { return(ptr, size) }
        }
    }
}

// Logic Contract V1
contract LogicV1 {
    uint256 public counter;

    function increment() external {
        counter += 1;
    }
}

// Logic Contract V2 (Vulnerable)
contract LogicV2 {
    address public counter; // 🔴 Potential Type Confusion: The type of 'counter' has changed from uint256 to address.

    function setCounter(address _addr) external {
        counter = _addr; // This overwrites storage slot, potentially breaking assumptions of Proxy.
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


