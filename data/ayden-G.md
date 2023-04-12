https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L94#L100
Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past

use assembly to check for address(0) save more gas

```solidity
error ZeroAddress();
function assembly_notZero(address toCheck) public pure returns(bool success) {
    assembly {
        if iszero(toCheck) {
            let ptr := mload(0x40)
            mstore(ptr, 0xd92e233d00000000000000000000000000000000000000000000000000000000) // selector for `ZeroAddress()`
            revert(ptr, 0x4)
        }
    }
    return true;
}
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484#L507
Optimize code logic by reducing the check for zero address when dealing with 'baseToken', storing the length of 'tokenIds' as a local variable, and moving the 'i++' incrementation inside the 'unchecked' statement.

```solidity
        // ~~~ Checks ~~~ //

        if((baseToken == address(0))){
            //eth
            if(msg.value != baseTokenAmount)  revert InvalidEthAmount();
        }else{
            //erc20
            if(msg.value > 0) revert InvalidEthAmount();
            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);
        }

        // ~~~ Interactions ~~~ //

        // transfer the nfts from the caller
        uint256 tokenIdsLength = tokenIds.length;
        for (uint256 i = 0; i < tokenIdsLength;) {
            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
            unchecked{i++;}
        }

        // emit the deposit event
        emit Deposit(tokenIds, baseTokenAmount);
    }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L221
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L306
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L393
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514
If a function is only called externally, it can be marked as 'external' instead of 'public' to save more gas

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225
The check for the caller sending 0 ETH when the base token is not ETH can be placed at the beginning of the function, which can reduce calculations earlier in the code in case of an exception

```solidity
 function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
    {
        // ~~~ Checks ~~~ //
+       if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
        // calculate the sum of weights of the NFTs to buy
        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

        // calculate the required net input amount and fee amount
        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

        // check that the caller sent 0 ETH if the base token is not ETH
-       if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

```
