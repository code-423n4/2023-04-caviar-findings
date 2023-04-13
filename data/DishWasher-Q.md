# Quality Analysis

## Logic, Grammar and Spelling in comments

### EthRouter.sol
- remove second "the" in comment in buy() (https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L105)

### PrivatePool.sol
- amount instead of aount (https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L251)
 
- netOutputAmount in sell() is excluse of fees not inclusive (https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L299, https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L708, https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L711)

- description of change function "The sum of the caller's NFT weigths must be **greater** than or equal to the sum of the output pool NFTs weigths." instead of **less** than (https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L376-L377)


## Other QAs

- if there was money in the eth router before calling sell, the seller will receive too much money (https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L208)


- It should be possible to pay fees with the excess amount of money one has put in through more valuable input NFTs compared to output NFTs, instead of only by sending ETH (https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L428-L429)
