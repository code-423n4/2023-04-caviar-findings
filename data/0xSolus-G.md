[L#271](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L271) - Redundant code: `buy()` calculates `_getRoyalty()` twice unnecessarily as the 
operation can be done in the first loop, avoiding the second one in L#272 make sure to optimize. 
Gas costs old: avg -> 70884, median -> 74037, max -> 158864.  
Gas costs new: avg -> 70311 , median -> 73894 , max -> 147884.  
Recommendation:   
```
function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
  public
  payable
returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
{
  // ~~~ Checks ~~~ //

  // calculate the sum of weights of the NFTs to buy
  uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

  // calculate the required net input amount and fee amount
  (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

  // check that the caller sent 0 ETH if the base token is not ETH
  if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

  // ~~~ Effects ~~~ //

  // update the virtual reserves
  virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
  virtualNftReserves -= uint128(weightSum);

  // ~~~ Interactions ~~~ //

  // calculate the sale price (assume it's the same for each NFT even if weights differ)
  uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
  uint256 royaltyFeeAmount = 0;
  for (uint256 i = 0; i < tokenIds.length; i++) {
    // transfer the NFT to the caller
    ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

    if (payRoyalties) {
      // get the royalty fee for the NFT
      (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

      // transfer the royalty fee to the recipient if it's greater than 0
      if (royaltyFee > 0 && recipient != address(0)) {
        if (baseToken != address(0)) {
          ERC20(baseToken).safeTransfer(recipient, royaltyFee);
        } else {
          recipient.safeTransferETH(royaltyFee);
        }
      }
      // add the royalty fee to the total royalty fee amount
      royaltyFeeAmount += royaltyFee;
    }
  }

  // add the royalty fee amount to the net input aount
  netInputAmount += royaltyFeeAmount;

  if (baseToken != address(0)) {
    // transfer the base token from the caller to the contract
    ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);

    // if the protocol fee is set then pay the protocol fee
    if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
  } else {
    // check that the caller sent enough ETH to cover the net required input
    if (msg.value < netInputAmount) revert InvalidEthAmount();

    // if the protocol fee is set then pay the protocol fee
    if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

    // refund any excess ETH to the caller
    if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);
  }

  // emit the buy event
  emit Buy(tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);
}
```

