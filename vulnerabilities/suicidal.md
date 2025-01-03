# Suicidal Contract Vulnerabilitiy
The `selfdestruct` opcode, known as suicide, enables a  CoB to terminate itself, releasing the Ether held in its accounts. This feature is typically utilized in emergencies. A SC is suicidal if it fails to secure the usage of `selfdestruct` opcode properly. 
As a result of attacks abusing this feature, the contract becomes irresponsive because the code section is permanently deleted resulting in a DoS scenario. Besides, the transfer of the contract's Ether might also be abused to drain the assets. All the funds associated with the SC are transferred to the designated beneficiary account without triggering fallback functions. The beneficiary can either be an existing account or a non-existing one.

## Toy Example
contract Self-destructExample {
    // Address of the contract owner
    address payable public contractOwner;

    // Constructor sets the contract deployer as the owner
    constructor() {
        contractOwner = payable(msg.sender);
    }
    // Vulnerable function that allows anyone to destroy the contract
    function destroyContract() public {
        // No access control: Any user can call this function and destroy the contract
        selfdestruct(payable(msg.sender)); 
        // The contract's remaining balance is sent to the caller (msg.sender)
    }
    // Function to allow deposits to the contract
    function deposit() public payable {
        // Accepts Ether deposits to the contract balance
    }
}

## A Real-World Incident Involving Self Destruction
In November 2017, the Parity multi-signature wallet experienced a catastrophic vulnerability stemming from an improperly secured use of the `selfdestruct` opcode in its shared library contract. Multi-signature wallets created by Parity relied on a library contract for core functionality. However, the developers failed to adequately secure the initialization and ownership of this library contract.

A user, exploiting this oversight, was able to invoke the `initWallet` function on the library contract directly. This function, designed for wallet initialization, had not been disabled after the library’s deployment. As a result, the user became the owner of the library contract. With ownership rights, the user executed the `kill` function (now known as `selfdestruct`), which permanently destroyed the library contract.

The destruction of the library contract had far-reaching consequences. All multi-signature wallets deployed by Parity that depended on this library lost access to critical functionality. Specifically, the code associated with the library was permanently removed from the blockchain, rendering calls to these wallets’ functions ineffective. Consequently, approximately 513,774.16 Ether—valued at around $150 million at the time—was frozen in the wallets, as there was no longer any executable code to facilitate transactions or fund recovery.

This incident underscores the risks of improper handling of the `selfdestruct` opcode and the critical importance of securing library contracts. Developers must ensure that initialization functions are restricted after deployment and that access control mechanisms are rigorously enforced to prevent unauthorized invocations of destructive operations.


## References 

[1] https://medium.com/@web3author/parity-wallet-hack-demystified-all-you-need-to-know-91b8dcb5b81

