# Call-Stack Depth Limit
Due to Ethereum's 1024-frame stack limit per transaction, attackers can use recursive calls to fill the stack and prevent following calls from succeeding. This is known as the Call-stack Depth issue. By raising the gas price for external calls, EIP-150 somewhat reduced the impact of these attacks, and EIP-214 reduced the dangers associated with external interactions by introducing STATICCALL for read-only calls. The stack depth limit, however, is still in effect. The EVM's architecture, which was created to avoid excessive resource usage and preserve system efficiency, is inherently limited. Network performance could be harmed by changing this restriction, and contracts with intricate or sequential calls are still vulnerable to this problem.

## Toy Example

## Real World Example

## References
