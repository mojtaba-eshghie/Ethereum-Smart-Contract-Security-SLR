# Resource Exhaustion
The gas mechanism in EVM is used to determine how much it costs to execute smart contracts. This system should guarantee that each instruction's execution cost is calculated according to the resources it uses, including CPU, RAM, and I/O. Nevertheless, some processes, such as BLOCKHASH and BALANCE, which require access to blockchain-stored data, are substantially underpriced compared to their actual computational and storage impact due to inconsistent gas pricing for specific EVM instructions.  

Exploitation of smart contracts is enabled by inconsistencies in resource metering, allowing attackers to create low-throughput contracts that consume excessive computational power at minimal cost. This leads to a Resource Exhaustion Attack, where crafted transactions cause severe delays, preventing network nodes from staying synchronized with the blockchain. 

## Toy Example

## Real World Example

## References
