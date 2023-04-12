L-01:EthRouter#buy/sell does not determine whether royaltyRecipient is address(0)
If `getRoyalty()` returns royaltyFee>0, but `royaltyRecipient==address(0)` this will transfer the funds to address(0)
Suggest adding judgment:

```solidity
    function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
...
                if (payRoyalties) {
                    uint256 salePrice = inputAmount / buys[i].tokenIds.length;
                    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
                        // get the royalty fee and recipient
                        (uint256 royaltyFee, address royaltyRecipient) =
                            getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);

-                       if (royaltyFee > 0) {
+                       if (royaltyFee > 0 && royaltyRecipient!=address(0)) {
                            // transfer the royalty fee to the royalty recipient
                            royaltyRecipient.safeTransferETH(royaltyFee);
                        }
                    }
                }    
```

L-02:setProtocolFeeRate() does not limit the maximum value, there is a risk of malicious amplification of the fee
It is recommended that this should not exceed, for example, 5%

```solidity
contract Factory is ERC721, Owned {
...
    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
+       if (_protocolFeeRate > 5_000) revert FeeRateTooHigh();  
        protocolFeeRate = _protocolFeeRate;
    }
```