# Frozen Ether
Frozen Ether (or Locked Money) is a vulnerability in smart contracts where funds (such as Ether) become locked within the contract, rendering them non-transferable or non-retrievable. This issue typically arises due to flaws in the design or implementation of the smart contract code, where unexpected or rare conditions prevent functions like refunds or fund transfers from executing. It can result from logical limitations, access control errors, or reliance on external functions that fail under specific circumstances. For example, in the Akutars NFT incident, a flaw in the transaction control mechanism permanently locked over $34 million worth of Ether. Similarly, a critical library contract was accidentally deleted in the Parity Multisig Wallet incident, freezing over 580,000 Ether. This vulnerability highlights the importance of secure design and rigorous testing of smart contracts to prevent such issues.

## Toye Example
```Solidity
pragma solidity ^0.8.0;

contract FrozenEtherExample {
    mapping(address => uint256) public balances;
    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "No balance to withdraw");

        balances[msg.sender] = 0;

        // Vulnerable line: If this call fails, the Ether is stuck in the contract
        (bool sent, ) = msg.sender.call{value: amount}("");
        require(sent, "Failed to send Ether");
    }
}
```
## Real World Example
On April 22, 2022, a vulnerability was identified in a series of smart contracts related to online auctions, resulting in approximately $34 million worth of Ether being locked within these contracts. The contract under discussion, AkuAuction or a similar one from this series, was designed to manage bids and refunds in online auctions. While the analyzed contract might not be the exact one exploited in the incident, its design shares structural similarities with those affected, highlighting common flaws that led to this significant loss. The identified vulnerabilities and reasons behind the Frozen Ether issue are detailed below.
### Vulnerability Points
The contract exhibits three critical vulnerability points that collectively resulted in the locking of users’ funds:

1. Refund Progress Check
At the beginning of the processRefunds function, the following condition is used to verify the status of refunds:
```Solidity
require(_refundProgress < _bidIndex, "Refunds already processed");
```
 This condition checks if all refunds have been processed. The _refundProgress variable tracks the number of refunds processed, while _bidIndex represents the total number of bids. If, for any reason (such as gas limitations), _refundProgress is not updated, this condition will always evaluate as true, effectively halting further refund processing and leading to funds being locked within the contract.
2. Gas Limit in the Refund Loop
In the same function, the loop responsible for processing refunds is written as follows:
```Solidity
for (uint256 i=_refundProgress; gasUsed < 5000000 && i < _bidIndex; i++) {
    bids memory bidData = allBids[i];
    if (bidData.finalProcess == 0) {
        uint256 refund = (bidData.price - price) * bidData.bidsPlaced;
        if (refund > 0) {
            (bool sent, ) = bidData.bidder.call{value: refund}("");
            require(sent, "Failed to refund bidder");
        }
    }
    _refundProgress++;
}
```
 This loop attempts to process refunds within a gas limit of 5000000. If the transactions are complex or the gas available is insufficient, the loop may fail to complete all refunds. Consequently, the _refundProgress variable may not update entirely, perpetuating the condition in the previous vulnerability and preventing the remaining refunds from being processed.
3. Use of call for Ether Transfers
The contract uses the call function to send Ether to bidders:
```Solidity
(bool sent, ) = bidData.bidder.call{value: refund}("");
require(sent, "Failed to refund bidder");
```
 Using call for transferring Ether introduces several risks:
If the recipient is a smart contract and its fallback or receive function consumes excessive gas or encounters an error, the transfer fails.
A failed transfer interrupts the refund process, leaving _refundProgress incomplete and locking funds within the contract.
### Combined Impact
The combination of these vulnerabilities creates a situation where:

- The refund progress check condition (_refundProgress < _bidIndex) perpetually blocks further processing.

- The refund loop halts prematurely due to gas limitations or failed transactions.
- 
- A lack of fallback mechanisms leaves locked funds irrecoverable, causing substantial financial losses.

## References
[1]   https://decrypt.co/98530/aku-ethereum-nft-launch-ends-with-34m-locked-in-flawed-smart-contract

[2] https://blog.openzeppelin.com/on-the-parity-wallet-multisig-hack-405a8c12e8f7

[3] https://etherscan.io/address/0xf42c318dbfbaab0eee040279c6a2588fa01a961d#code

