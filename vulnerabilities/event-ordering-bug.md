# Event-Ordering Bug
If distinct sequences of events, such as transactions and invocations, lead to disparate outcomes in a contract, it is identified as an EO bug.  This inconsistency arises because the order in which events are processed can impact the contract's state or behavior. 
If undesired orders are allowed by the  require statements in the contract, it can lead to unpredicted or unintended results, creating a potential vulnerability. 
## Toy Example
```Solidity

contract VulnerableCasino {
    uint256 public prize;
    address public winner;

    function setWinner(address _winner) public {
        winner = _winner; // State-changing operation
    }

    function sendPrize() public {
        require(winner != address(0), "Winner not set"); // Depends on previous state change
        payable(winner).transfer(prize); // Transfers the prize to the winner
    }
}

```
## Real World Example 
In 2017, a Casino smart contract suffered from an event ordering (EO) bug, leading to incorrect payouts and potential financial losses​
. The contract used an off-chain oracle (Oraclize API) to generate random numbers to determine winners. The issue arose because bets were accepted and processed asynchronously, meaning oracle callbacks could return in an unexpected order. If multiple players placed bets at the same time, and the oracle responses were processed out of order, some winners would not receive their payouts if the contract's balance was insufficient after processing an earlier payout. This event ordering flaw made the game unfair and exploitable, as players could manipulate the bet timing to increase their chances of receiving payouts first.

```Solidity
bytes32 oid = oraclize_query(...);  
bets[oid] = msg.value;
players[oid] = msg.sender;

if (parseInt(result) % 200 == 42)
    players[myid].send(bets[myid] * 100);   

```
This flaw becomes evident when multiple players place bets simultaneously—since bets are stored in a mapping (bets[oid]) and referenced via oid (oracle query ID), the contract assumes that payouts will be processed in the same order as bets were placed, which is not guaranteed. If a later bet’s oracle response is processed first, the contract may incorrectly distribute payouts, potentially depleting funds before earlier bets are resolved, leading to unfair results.


## References
[1] Kolluri, A., Nikolic, I., Sergey, I., Hobor, A., & Saxena, P. (2019, July). Exploiting the laws of order in smart contracts. In Proceedings of the 28th ACM SIGSOFT international symposium on software testing and analysis (pp. 363-373).
