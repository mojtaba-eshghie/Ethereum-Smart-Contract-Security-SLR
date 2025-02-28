# DoS Attack via Owner Account

Many smart contracts designate an owner account with privileged control over critical functions such as fund withdrawals, parameter modifications, and contract upgrades. If this account is not properly secured, attackers could exploit vulnerabilities like private key exposure, phishing attacks, or weak authentication mechanisms to gain unauthorized access. This could result in severe consequences, such as permanently locking Ether within the contract, unauthorized fund transfers, disabling essential functions, or even triggering self-destruct mechanisms if implemented. To prevent such risks, developers must implement multi-signature authentication, timelocks for critical operations, and role-based access controls to limit the potential damage from compromised ownership.

## Toy Example
The VulnerableCrowdsale contract lacks access control, allowing anyone to become the owner (setOwner), change the status (setStatus), and withdraw all funds (withdraw) if they take control. This makes it vulnerable to ownership hijacking, fundraising manipulation, and fund theft.


```Solidity
contract VulnerableCrowdsale {
    uint256 public raised;
    uint256 public goal = 5 ether;
    uint256 public status; // 0 = ongoing, 1 = success, 2 = failed
    address public owner;
    mapping(address => uint256) public deposits;

    constructor() {
        owner = msg.sender;
    }

    function invest() public payable {
        deposits[msg.sender] += msg.value;
        raised += msg.value;
    }

    function setOwner(address newOwner) public {
        owner = newOwner;  // No access control! Anyone can become the owner.
    }

    function setStatus(uint256 newStatus) public {
        require(newStatus == 1 || newStatus == 2, "Invalid status");
        status = newStatus;  // No access control! Anyone can change status.
    }

    function withdraw() public {
        require(status == 1, "Fundraising not successful");
        payable(owner).transfer(raised);  // If attacker is the owner, they steal all funds.
    }
}

```

## Real World Example
Crowdsale smart contract, a real example of an account owner attack, suffered from a DoS attack via the owner account, which ultimately allowed an attacker to steal all raised funds. The attack was possible due to missing access control mechanisms in key functions of the contract. The vulnerabilities allowed any user to change the contract's ownership, update its funding status, and withdraw funds. As a result, investors lost their deposits, and the fundraiser failed. Below is a detailed breakdown of how this attack worked.

 One of the biggest flaws in the contract was the lack of access control in the setOwner() function. Normally, only the current owner should be able to assign a new owner. However, in this contract, any user could claim ownership.
```Solidity 
function setOwner(address newOwner) public {
    owner = newOwner;  // No access control! Anyone can become the owner.
}
```
Since there is no required (msg.sender == owner) check, an attacker could call setOwner() and replace the contract's legitimate owner with their address. This flaw gave the attacker complete control over the contract.
After gaining ownership, the attacker could manipulate the contract’s state using another weakly protected function: setStatus(). This function determines whether the fundraiser has succeeded or failed, and its result affects who can withdraw funds.
```Solidity
function setStatus(uint256 newStatus) public {
    require((newStatus == 1 && raised >= goal) ||
            (newStatus == 2 && raised < goal && now >= closeTime));
    status = newStatus;  // No owner check! Anyone can change the status.
}
```
Since there is no restriction on who can call setStatus(), an attacker could force the contract to mark the fundraiser as successful (status = 1) even if the original owner did not intend to do so. This was a crucial step in the attack.
The final and most damaging flaw was in the withdrawal mechanism. After the fundraiser was marked as successful, the owner was allowed to withdraw all funds.
```Solidity
function withdraw() public {
    require(status == 1);
    owner.transfer(raised);  //If an attacker is the owner, they can steal all funds.
}
```
Since the attacker had already taken ownership, they could now call withdraw() and transfer all raised funds to their own address. This effectively emptied the contract and left legitimate investors with nothing.

## Referen
[1] https://github.com/LeadcoinNetwork/crowdsale-smart-contract
