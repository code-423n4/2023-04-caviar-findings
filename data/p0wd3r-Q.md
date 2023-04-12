# Check if tokenIds is empty at the beginning of the buy function.

PrivatePool.sol L211

```
function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof) {
...
emit Buy(tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);
}
```

The `buy` function does not check if `tokenIds` is empty. If it is empty, the function will continue to execute and eventually emit an event that has no effect and should not be emitted.


# Replace `transferFrom` with `safeTransferFrom`

Factory.sol L115

```
 if (_baseToken == address(0)) {
            // transfer eth into the pool if base token is ETH
            address(privatePool).safeTransferETH(baseTokenAmount);
        } else {
            // deposit the base tokens from the caller into the pool
            ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);
        }
```

Replace `transferFrom` with `safeTransferFrom`, as some non-standard ERC20 tokens do not revert on failed transfers.