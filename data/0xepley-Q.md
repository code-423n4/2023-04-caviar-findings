In this line https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L130 owner is updating _privatePoolMetadata which should be in two step instead of 1.

Same is the case with this one https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135

In the line https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L563 there is a Typo in the comment the should should be `check that the fee rate is greater than 50%`

There isn't any kind of zero address check in `create` function of `factory.sol` here is the link https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L73

In this function there is a chance that owner will input `amount` as zero so gas will be lost, consider adding `amount>0` check here https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L148

In the `deposit` function of `EthRouter.sol` there isn't any kind of zero address check on `address nft`
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L221