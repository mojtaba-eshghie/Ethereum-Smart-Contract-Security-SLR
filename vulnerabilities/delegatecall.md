# Dangerous Delegatecall
This vulnerability was initially identified during an attack on the Parity wallet, leveraging the delegate-call opcode in the EVM. The delegate-call opcode permits the CoB of a callee contract to be integrated into the CoB of the calling contract, facilitating code reuse. 
This might lead to a malicious callee contract gain direct access to manipulate the state variables of the calling contract.

## Toy Example


## Real-World Example

## References

[] 
