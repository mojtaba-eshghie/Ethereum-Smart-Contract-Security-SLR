# Type Mismatch
In smart contracts, business logic errors—also referred to as accounting errors—occur when financial calculations are inconsistent, inaccurate, or disorganized because of type errors, misinterpreted or mismanaged units token unit, which can result in %fund mismanagement and 
exploitation. 

## Toy Example
 ```Solidity
contract LiquidityPool {
    uint public totalLiquidity = 1000;

    function addLiquidity(uint userAmount) public returns (uint liquidity) {
        liquidity = (userAmount * totalLiquidity) / 2;  // Incorrect formula! Can cause unfair liquidity distribution.
        totalLiquidity += liquidity;
    }
}

```
## Real World Example
In the Vader Protocol project, a critical Type Mismatch vulnerability was identified in the calcLiquidityUnits function, which was responsible for calculating liquidity based on token inputs. The function incorrectly combined units of different tokens without properly aligning them, leading to incorrect liquidity allocations. This vulnerability was discovered in 2023 and caused a financial loss of $57 million for users. Despite undergoing multiple rounds of manual and automated audits, this issue remained undetected and became an example of a high-risk vulnerability in smart contracts.
```Solidity
 function calcLiquidityUnits(uint b, uint B, uint t, uint T, uint P) external view returns (uint){
    uint part1 = (t * B);
    uint part2 = (T * b);
    uint part3 = (T * B) * 2;
    uint _units = (((P * part1) + part2) / part3);
    return (_units);
}
```
This function was designed to calculate the amount of liquidity generated from the base tokens and the tokens added to the pool. However, the primary error occurred when the variables part1 and part2 were combined without properly aligning their units, resulting in a Type Mismatch. The main error occurred in the calculation of _units. The variable part1, which was the product of the new token (t) and the existing token reserve (B), had a different unit compared to part2, which was the product of the old token (T) and the added amount (b). On the other hand, the denominator, defined in part3, was calculated as (T * B) * 2. When part1 and part2 were added together, the mismatch in their units caused an error in the final _units value. This error was not detected due to Solidity's inability to verify and manage the compatibility of variable units.

## References
[1] Zhang, B. (2024, April). Towards Finding Accounting Errors in Smart Contracts. In Proceedings of the IEEE/ACM 46th International Conference on Software Engineering (pp. 1-13).

[2] https://code4rena.com/reports/2021-04-vader
