Solmate’s SafeTransferLib doesn’t check the ERC20 contract code size:

In contrast to OpenZeppelin's SafeTransferLib, Solmate’s SafeTransferLib is often used to interact with non-compliant ERC20 tokens, and does not check whether the ERC20 contract exists. And thus, it always returns true for cases where the token is not deployed and there is no code inside the mentioned address. There exist several cases for this issue:

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L256
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L259
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L265
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L268
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L279
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L281
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L346
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L348
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L359
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L362
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L368
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L423
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L426
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L432
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L436
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L502
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L524
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L112
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L150
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L123
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L142
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L190
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L208
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L291
