**G-1**

The PrivatePool logic can be significantly simplified if ETH logic is removed and ERC-20 tokens only are used. Instead of using pure ETH, the Wrapped ETH (WETH) can be used.

**G-2**

The EthRouter already has functions implemented for working with royalties for public pools. It is possible to avoid duplicating this logic in PrivatePool and instead pay all royalties from EthRouter.


**G-3**

PrivatePoolMetadata implements the full logic of generating JSON and SVG files, including converting them to Base64. This is excessive. It is sufficient to store only the data for building the files and images on the blockchain, and move the logic for their construction outside the blockchain to the frontend. This will significantly save gas during contract deployment and also provide flexibility if the SVG image needs to be improved or modified in the future.


**G-4**

The PrivatePool function sumWeightsAndValidateProof() can be made `internal`, which will significantly reduce gas when called from the buy(), sell(), and change() functions.

It is important that the calldata/memory modifier for the array parameters in the internal function sumWeightsAndValidateProof() matches the modifier for the passed array from the buy(), sell(), and change() functions, otherwise there will be no gas savings.

It is ideal if `tokenIds` and `tokenWeights` in all functions has `calldata` modifier, with the buy(), sell(), change(), and execute() functions being `external`, and the sumWeightsAndValidateProof() function being `internal`:
```
function sumWeightsAndValidateProof(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof
) internal

...

function buy(
        uint256[] calldata tokenIds, 
        uint256[] calldata tokenWeights, 
        MerkleMultiProof calldata proof
) external

...

function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        ...
) external

...

function change(
        uint256[] calldata inputTokenIds,
        uint256[] calldata inputTokenWeights,
        uint256[] calldata outputTokenIds,
        uint256[] calldata outputTokenWeights,
        MerkleMultiProof calldata inputProof,
        MerkleMultiProof calldata outputProof
        ...
) external

...

function execute(
        address target, 
        bytes calldata data
) external
```

