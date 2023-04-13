## Gas Optimization Findings
| Number |Issues Details |
|:--:|:-------|
|[GAS-01]| Streamline if/else statements to save gas
|[GAS-02]| Streamline for loops to save gas
***

## |[GAS-01]| Streamline if/else statements to save gas
We can move some if/else checks in order to save gas by skipping or eliminating unnecessary operations
PrivatePool.sol
Move if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
to the first line of the buy function
BEFORE
function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
    public
    payable
    returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
    {
    
    uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
    (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
AFTER
if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
    (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

Move if(msg.value < netInputAmount) before any transfering of tokens and then reverting to save gas.
BEFORE
if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
        } else {
            if (msg.value < netInputAmount) revert InvalidEthAmount();
AFTER
if (msg.value < netInputAmount) revert InvalidEthAmount();
if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

*** 
## |[GAS-02]| Streamline for loops to save gas

PrivatePool.sol

The following two for loops iterate over the same array and do similar tasks and can be combined into one to save a lot of gas, especially if transfering a lot of NFTs.

for (uint256 i = 0; i < tokenIds.length; i++) {
            // transfer the NFT to the caller
            ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

            if (payRoyalties) {
                // get the royalty fee for the NFT
                (uint256 royaltyFee,) = _getRoyalty(tokenIds[i], salePrice);

                // add the royalty fee to the total royalty fee amount
                royaltyFeeAmount += royaltyFee;
            }
        }
for (uint256 i = 0; i < tokenIds.length; i++) {
    (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

    if (royaltyFee > 0 && recipient != address(0)) {
        if (baseToken != address(0)) {
                ERC20(baseToken).safeTransfer(recipient, royaltyFee);
            } else {
            recipient.safeTransferETH(royaltyFee);
            }
            }
    }