https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L623

In privatepool.sol at flashloan https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L648 
instead of transfering the nfts to the why not the the borrower transfers it to the pool and then the pool verifies that the owner of the nft is its address. just a minor suggestion