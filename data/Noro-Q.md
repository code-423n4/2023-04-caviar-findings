# Check that `tokenIds.lenght == tokenWeights.lenght` before action in `PrivatePool.sumWeightsAndValidateProof`

If `tokenIds.lenght > tokenWeights.lenght`, for loop will iterate until `i > tokenWeights.lenght`, and then revert with error : ***Index out of bounds*** . This costs gas.

If `tokenWeights.lenght < tokenIds.lenght`, all leafs will not be taken in account, and consequently, MerkleProofLib.verifyMultiProof() returns false, and transaction will revert with error : ***InvalidMerkleProof()***.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L661

## Recommendation
Add a check before entering for loop
 
``
        require(tokenIds.length == tokenWeights.length);
``