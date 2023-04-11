# Low Severity Findings

1. The doc says:
 > Initially the protocol fee rate will be set to be 0% however it may be increased in the future, with advanced notice.

But there is no `timelock` function [implemented](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141) in the smart contracts to make sure that the promise of `advanced notice` will kept.
2. Though most of the pool parameters can be changed later, `changeFee` remains immutable. This option could be nice for the pool owners. 

3. The royalties, if enabled, requires the NFT to be listed in manifold.xyz. If its not listed there, the royalty program will fail. 
4. [setVirtualReserves](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538) function doesnt check if the new values are zero or not. If `virtualBaseTokenReserves` is zero then, `buyQuote` and `sellQuote` will return zero. If `virtualNftReserves` is zero then, `buyQuote` will revert.
5. The extra msg.value sent when calling the [flashLoan](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L623) function doesnt get returned to the user. Though this refund is done when the `buy` and `change` functions are called.
6. Pool owner can [change](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L550) the merkle root after increasing the weights of existing tokens in the pool, so that buyers need to pay much higher than what sellers got it when they sold it. 
7. The stolen NFT tokens are not [not allowed](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L316-L318) to be sold into the pool. But there is no such restriction to use stolen tokens with `flashLoan` function. This seems counterintuitive. 
8. `flashLoan` usage of a token only pays `pool fee` which goes to the pool owner. But `royaltyFees` are not paid. This seems not right.