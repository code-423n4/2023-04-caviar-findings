1. The `Factory.create` function is susceptible to re-entrancy as it performs a `_safeMint` before initializing the pool.
    ```solidity
    function create(
        ...
    ) public payable returns (PrivatePool privatePool) {
        // ...
        privatePool = PrivatePool(payable(privatePoolImplementation.cloneDeterministic(_salt)));

        // mint the nft to the caller
        _safeMint(msg.sender, uint256(uint160(address(privatePool))));

        // initialize the pool
        privatePool.initialize(...);
        // ...
    }
    ```
2. The `Factory.create` function performs plain transfer of funds instead of calling the `deposit` function. This way the `Deposit` event is not emitted.
    ```solidity
    function create(
        ...
    ) public payable returns (PrivatePool privatePool) {
        // ...
        privatePool.initialize(...);
        
        if (_baseToken == address(0)) {
            // transfer eth into the pool if base token is ETH
            address(privatePool).safeTransferETH(baseTokenAmount);
        } else {
            // deposit the base tokens from the caller into the pool
            ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);
        }

        // deposit the nfts from the caller into the pool
        for (uint256 i = 0; i < tokenIds.length; i++) {
            ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
        }

        // emit create event
        emit Create(address(privatePool), tokenIds, baseTokenAmount);
    }
    ```
3. There is no fee cap on the `Factory.setProtocolFeeRate` function. A value greater then 10000 can break the fee calculations in private pool. Consider validating that the input is less than 10000.
    ```solidity
    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }
    ```
4. `Factory.tokenId` does not validate the input `id` parameter. Consider validating that the `id` exist and the respective pool is created by the factory.
    ```solidity
    function tokenURI(uint256 id) public view override returns (string memory) {
        return PrivatePoolMetadata(privatePoolMetadata).tokenURI(id);
    }
    ```
5. Once initialization is done the `PrivatePool.feeRate` variable can never be changed. Consider adding an `owner` restricted function to update `feeRate`.
    
6. In `buy` and `sell` functions consider validating that the `length` of all input arrays are equal (`tokenIds` & `tokenWeights`).
   
7. Consider adding a check in `PrivatePool.sumWeightsAndValidateProof` function to validate that every element of `tokenWeights` array is greater than or equal to `1e18`.
    ```solidity
    function sumWeightsAndValidateProof(
        uint256[] memory tokenIds,
        uint256[] memory tokenWeights,
        MerkleMultiProof memory proof
    ) public view returns (uint256) {
        // ...
        for (uint256 i = 0; i < tokenIds.length; i++) {
            require(tokenWeights[i] >= 1e18);       <------------
            // create the leaf for the merkle proof
            leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

            // sum each token weight
            sum += tokenWeights[i];
        }
        // ...
    }
    ```
8. In the `PrivatePoolMetadata.tokenURI` function consider using `Strings.toHexString(address(tokenId)))` for the `name` field.
    ```solidity
    function tokenURI(uint256 tokenId) public view returns (string memory) {
        // forgefmt: disable-next-item
        bytes memory metadata = abi.encodePacked(
            "{",
                '"name": "Private Pool ',Strings.toString(tokenId),'",',
                '"description": "Caviar private pool AMM position.",',
                '"image": ','"data:image/svg+xml;base64,', Base64.encode(svg(tokenId)),'",',
                '"attributes": [',
                    attributes(tokenId),
                "]",
            "}"
        );

        return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));
    }
    ```