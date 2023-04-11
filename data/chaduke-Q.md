QA1. Zero address check is recommended for the input: 

[https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L131](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L131)

[https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135-L137](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135-L137)

QA2.A range check is recommended for the input:
[https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141-L143](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141-L143)

QA3. One should check whether tokenIds.length == tokenWeights.length.

[https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L661-L687](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L661-L687)

QA4. Maybe ``bytes.concat`` is not needed here and the calculation can be simplified:

```diff
- leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
+ leafs[i] = keccak256(abi.encode(tokenIds[i], tokenWeights[i]));

```

QA5. There is an error in the NatSpec of ``change()``:

[https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L375-L377](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L375-L377)

Mitigation: the correct should be 
```diff
- The sum of the caller's NFT weights must be less than or equal to the sum of the
    /// output pool NFTs weights.

+ The sum of the caller's NFT weights must be greater than or equal to the sum of the
    /// output pool NFTs weights.
```

QA6. The ``buy()`` function fails to verify that a private pool only supports ETH as base tokens.

[https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L129)

Mitigation: check whether the private pool only supports ETH as base tokens:
```diff
function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
        // check that the deadline has not passed (if any)
        if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }

        // loop through and execute the the buys
        for (uint256 i = 0; i < buys.length; i++) {
            if (buys[i].isPublicPool) {
                // execute the buy against a public pool
                uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(
                    buys[i].tokenIds, buys[i].baseTokenAmount, 0
                );

                // pay the royalties if buyer has opted-in
                if (payRoyalties) {
                    uint256 salePrice = inputAmount / buys[i].tokenIds.length;
                    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
                        // get the royalty fee and recipient
                        (uint256 royaltyFee, address royaltyRecipient) =
                            getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);

                        if (royaltyFee > 0) {
                            // transfer the royalty fee to the royalty recipient
                            royaltyRecipient.safeTransferETH(royaltyFee);
                        }
                    }
                }
            } else {
                // execute the buy against a private pool
+               if(buys[i].pool.baseToken != address(0)) revert BaseTokenIsNotETH();
                PrivatePool(buys[i].pool).buy{value: buys[i].baseTokenAmount}(
                    buys[i].tokenIds, buys[i].tokenWeights, buys[i].proof
                );
            }

            for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
                // transfer the NFT to the caller
                ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
            }
        }

        // refund any surplus ETH to the caller
        if (address(this).balance > 0) {
            msg.sender.safeTransferETH(address(this).balance);
        }
    }
```

