# Resource Exhaustion
The gas mechanism in EVM is used to determine how much it costs to execute smart contracts. This system should guarantee that each instruction's execution cost is calculated according to the resources it uses, including CPU, RAM, and I/O. Nevertheless, some processes, such as BLOCKHASH and BALANCE, which require access to blockchain-stored data, are substantially underpriced compared to their actual computational and storage impact due to inconsistent gas pricing for specific EVM instructions.  

Exploitation of smart contracts is enabled by inconsistencies in resource metering, allowing attackers to create low-throughput contracts that consume excessive computational power at minimal cost. This leads to a Resource Exhaustion Attack, where crafted transactions cause severe delays, preventing network nodes from staying synchronized with the blockchain. 

## Toy Example
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ExtCodeSizeAttack {
    address public target;

    constructor(address _target) {
        target = _target;
    }

    function attack(uint256 iterations) public {
        for (uint256 i = 0; i < iterations; i++) {
            assembly {
                let size := extcodesize(target) // Repeated execution of EXTCODESIZE
                //`EXTCODESIZE` is a computationally expensive instruction that requires access to on-chain data.
                // Each execution of this instruction requires reading contract information, 
                // yet its gas cost was historically lower than the actual computational resource consumption.
                // By executing this instruction in a large loop, network nodes are forced to perform excessive processing,
                // leading to high computational overhead, network slowdown, and increased transaction costs for other users.
                 
            }
        }
    }
}

## Real World Example
In September 2016, an attack occurred on the Ethereum network due to the incorrect pricing of the EXTCODESIZE instruction in the EVM, which had a low gas cost despite the heavy computational requirements. Attackers used this flaw to execute DoS attacks, sending transactions with multiple EXTCODESIZE calls, overloading nodes with excessive disk access, leading to network slowdowns and higher gas costs. Additionally, improper use of EXTCODESIZE can cause security issues, like misidentifying addresses of smart contracts or externally owned accounts (EOA), potentially leading to vulnerabilities.

Attackers exploited the vulnerability by launching DoS attacks, where they sent transactions with numerous EXTCODESIZE calls to the network. This forced nodes to repeatedly read contract data from the disk without corresponding gas costs, causing increased computational load, slower block creation, and higher gas fees for regular users. Since attackers could launch these attacks with minimal cost, the impact on the network was severe despite the low expense of the transactions.

However, caution is still required when developing smart contracts, as improper use of EXTCODESIZE could lead to other security issues. For example, using EXTCODESIZE to check whether an address belongs to a smart contract or an EOA can yield incorrect results. During the contract constructor's execution, the code size is zero because the contract's code has not yet been fully deployed on the blockchain. Thus, checking the code size at this stage can lead to unexpected behaviors and security problems.

## References
[1] https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/extcodesize-checks/

[2] Perez, D., & Livshits, B. (2019). Broken metre: Attacking resource metering in EVM. arXiv preprint arXiv:1909.07220.
