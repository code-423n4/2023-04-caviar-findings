``` solidity 
function _getRoyalty(uint256 tokenId, uint256 salePrice)
        internal
        view
        returns (uint256 royaltyFee, address recipient)
    {
        // get the royalty lookup address
        address lookupAddress = IRoyaltyRegistry(royaltyRegistry).getRoyaltyLookupAddress(nft);

        if (IERC2981(lookupAddress).supportsInterface(type(IERC2981).interfaceId)) {
            // get the royalty fee from the registry
            (recipient, royaltyFee) = IERC2981(lookupAddress).royaltyInfo(tokenId, salePrice);

            // revert if the royalty fee is greater than the sale price
            if (royaltyFee > salePrice) revert InvalidRoyaltyFee();
        }
    }

```
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L778-L793

as I previously said, as we are setting the `_ntf` and lookupAddress is same ntf if not found. So we can manipulate royaltyfee by our malicious conract.