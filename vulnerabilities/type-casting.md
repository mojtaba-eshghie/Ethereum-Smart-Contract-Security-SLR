# Type casting
This vulnerability arises from improper type conversions in Solidity, particularly when casting between different integer types. For example, when a larger integer type like uint16 is cast to a smaller type like uint8, the excess bits are truncated, potentially leading to unexpected data loss. Another issue occurs when converting between signed and unsigned types of the same width, which can cause "signedness bugs." In such cases, negative values may be incorrectly interpreted as large positive numbers, or vice versa, leading to logical errors and unpredictable behavior in the contract.

## Tote Example
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TypeCastingVulnerabilityDemo {
    
    uint8 public smallNumber;

    // Function to set a large number and demonstrate truncation
    function setNumber(uint16 largeNumber) public {
        // Vulnerable line: Truncation occurs here
        smallNumber = uint8(largeNumber); // ⬤ This line is vulnerable to truncation
    }
}
```

## Real World Example
## REferences
