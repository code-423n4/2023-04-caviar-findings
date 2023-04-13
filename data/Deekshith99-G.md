## [G-1] structs can be closely packed
There are 2 instances of it
1st instance is at [EthRouter.sol#L48-L56](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L48-L56)
here `bool isPublicPool;` can be packed with ` address nft;` by this 
we can save 1 storage slot can save ~2100 of gas 

    struct Buy {
        address payable pool;
        address nft;         
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        uint256 baseTokenAmount;
        bool isPublicPool;
    }

`to`

    struct Buy {
        address payable pool; 
        address nft;         // 20 bytes
        bool isPublicPool;  // 1 byte 
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        uint256 baseTokenAmount;
       
    }

2nd instance is at [EthRouter.sol#L58-L67](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L58-L67)
here ` bool isPublicPool;` can be packed with  `address nft;` to save 1 storage slot
which can save ~ 2100 of gas

    struct Sell {
        address payable pool;
        address nft;
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        IStolenNftOracle.Message[] stolenNftProofs;
        bool isPublicPool;
        bytes32[][] publicPoolProofs;
    }
`to`

    struct Sell {
        address payable pool;
        address nft;      // 20 bytes
        bool isPublicPool;// 1 byte
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        IStolenNftOracle.Message[] stolenNftProofs;
        bytes32[][] publicPoolProofs;
    }
   
