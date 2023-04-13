## [QA-01] `msg.value` in loop

The change function in EthRouter is using `msg.value` in loop. Which means user has to send msg.value once and if there are ETHs in the contract they would be used instead.

```solidity
for (uint256 i = 0; i < changes.length; i++) {

            //...
            PrivatePool(_change.pool).change{value: msg.value}(
                _change.inputTokenIds,
                _change.inputTokenWeights,
                _change.inputProof,
                _change.stolenNftProofs,
                _change.outputTokenIds,
                _change.outputTokenWeights,
                _change.outputProof
            );

            // transfer NFTs to caller
            for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
                ERC721(_change.nft).safeTransferFrom(
                    address(this),
                    msg.sender,
                    _change.outputTokenIds[j]
                );
            }
}
```