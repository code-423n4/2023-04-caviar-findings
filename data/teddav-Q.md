# Router can only interact with ETH pools
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L129
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L247
When using `EthRouter` ’s `buy()` or `sell()` we can only interact with Pools that use ETH as `baseToken` 

Mitigation:
Consider adding ERC20 tokens. Only need to check if `baseToken` is `address(0)`
