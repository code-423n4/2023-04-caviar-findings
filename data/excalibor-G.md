https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441

The for loop at this line is suboptimal gas wise

we can optimize it by doing the for loop in the following way:
        ```uint256 inputTokenLength = inputTokenIds.length;
        for (uint256 i; i < inputTokenIds.length;) {
            ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
           unchecked{++i;}
        }```