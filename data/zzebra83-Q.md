# [L-01] sumWeightsAndValidateProof doesn't validate whether tokenId and tokenWeights are the same length

PrivatePool.sol#L661

The function assumes that the tokenIds and tokenWeights arrays have the same length. If the arrays have different lengths, it could result in unexpected behavior especially since function is used extensively within the contract. The function is also marked public so the unexpected behaviour could spread beyond the PrivatePool.sol contract. 

Two potentially bad scenarios:

1.  tokenWeights.length > tokenIds.length, in this case, extra data provided which will be ignored silently and might cause miscalculations on or off chain.

2. tokenIds.length > tokenWeights.length, the loop in sumWeightsAndValidateProof should revert because of
an index-out-of-bound error. We would save gas if we revert early instead of going through loop.

Potential Fix:

    function sumWeightsAndValidateProof(
        uint256[] memory tokenIds,
        uint256[] memory tokenWeights,
        MerkleMultiProof memory proof
    ) public view returns (uint256) {
        require(tokenIds.length == tokenWeights.length) // add this line @audit
        // if the merkle root is not set then set the weight of each nft to be 1e18
        if (merkleRoot == bytes32(0)) {
            return tokenIds.length * 1e18;
        }



# [G-01] Using unchecked blocks to save gas - Increments in for loop can be unchecked

For loops are used extensively throughout codebase(EthRouter.sol, Factory.sol, PrivatePool.sol) and significant gas savings can be achieved by incrementing loop counters using unchecked. 0.8 Solidity and above implements safety checks for all integer arithmetic, including overflow and underflow guards. 

However These increment operations never need to be checked for over/underflow because the variable will never reach the max number of uint256 or it will run out of gas before that happens. 

Here is how the loops look like now:

    for(uint256 i; i < 10; i++){
    //doSomething
    }

Here is how they should look like:

    for(uint256 i; i < 10;) {
    // loop logic
    unchecked { i++; }
    }

