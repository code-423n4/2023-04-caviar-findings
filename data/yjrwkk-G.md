### [G-01] Use != 0 instead of > 0 for unsigned integer comparison

When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.  

[src/EthRouter.sol#L121](https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L121)  
```
if (royaltyFee > 0) {
```
[src/EthRouter.sol#L141](https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L141)  
[src/EthRouter.sol#L188](https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L188)  
[src/EthRouter.sol#L290](https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L290)  
[src/Factory.sol#L87](https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L87)  
[src/PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L225)  
[src/PrivatePool.sol#L259](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L259)  
[src/PrivatePool.sol#L265](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L265)  
[src/PrivatePool.sol#L277](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L277)  
[src/PrivatePool.sol#L344](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L344)  
[src/PrivatePool.sol#L362](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L362)  
[src/PrivatePool.sol#L368](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L368)  
[src/PrivatePool.sol#L397](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L397)  
[src/PrivatePool.sol#L426](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L426)  
[src/PrivatePool.sol#L432](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L432)  
[src/PrivatePool.sol#L467](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L467)  
[src/PrivatePool.sol#L489](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L489)  
