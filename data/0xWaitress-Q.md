In the function PrivatePool::buy, tokenIds are looped twice, in the second loop the important task is to transfer royalty fee to the recipient, but the result of _getRoyalty can actually be skipped since it was called in the earlier loop

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L271-L283

```solidity
        if (payRoyalties) {
            for (uint256 i = 0; i < tokenIds.length; i++) {
                // get the royalty fee for the NFT
                (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice); @> audit

                // transfer the royalty fee to the recipient if it's greater than 0
                if (royaltyFee > 0 && recipient != address(0)) {
                    if (baseToken != address(0)) {
                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);
                    } else {
                        recipient.safeTransferETH(royaltyFee);
                    }
                }
```
save the needed amount in an arrays in the first loop

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238-L249

```solidity
+++ uint256[] memory royaltyFees;
+++ address[] memory recipients;
for (uint256 i = 0; i < tokenIds.length; i++) {
            // transfer the NFT to the caller
            ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

            if (payRoyalties) {
                // get the royalty fee for the NFT
                (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);
+++             royaltyFees[i] = royaltyFee; 
+++             recipients[i] = recipient; 
                // add the royalty fee to the total royalty fee amount
                royaltyFeeAmount += royaltyFee;
            }
        }

```