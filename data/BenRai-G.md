

# Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

There are 10 instances of this issue:

### File: src/Factory.sol
131: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner 
137: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner
143: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner 
150: function withdraw(address token, uint256 amount) public onlyOwner
 ### File: src/PrivatePool.sol

515: function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner
539: function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner
551: function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner
563: function setFeeRate(uint16 newFeeRate) public onlyOwner 
577: function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner 
588: function setPayRoyalties(bool newPayRoyalties) public onlyOwner


# Saving gas for the user by failing early
If the base token is not ETH there is now check if the user owns the amount of base tokens that are required to finish the transaction. I would recommend implementing such a check in the following functions as soon as possible, so in case the check fails the transaction fails early and saves the user gas in the process:
### File: src/PrivatePool.sol

212: function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

implement check after calculating the required Inputamound in line 223

386: function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount)
 
implement check after calculating the required fees in line 417

624: function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data)
        external
        payable
        returns (bool)
implement check after calculating the required flashFee in line 633


# public functions not called by the contract should be declared external instead
Changing the visibility saves gas.
There are 20 instances of this issue

### File: src/Factory.sol
73: function create(
        address _baseToken,
        address _nft,
        uint128 _virtualBaseTokenReserves,
        uint128 _virtualNftReserves,
        uint56 _changeFee,
        uint16 _feeRate,
        bytes32 _merkleRoot,
        bool _useStolenNftOracle,
        bool _payRoyalties,
        bytes32 _salt,
        uint256[] memory tokenIds, // put in memory to avoid stack too deep error
        uint256 baseTokenAmount
    ) public payable returns (PrivatePool privatePool)
137: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner 
143: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner
150: function withdraw(address token, uint256 amount) public onlyOwner
163: function tokenURI(uint256 id) public view override returns (string memory)
170: function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress)

### File: src/PrivatePool.sol
158: function initialize(
        address _baseToken,
        address _nft,
        uint128 _virtualBaseTokenReserves,
        uint128 _virtualNftReserves,
        uint56 _changeFee,
        uint16 _feeRate,
        bytes32 _merkleRoot,
        bool _useStolenNftOracle,
        bool _payRoyalties
    ) public
212: function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
302: function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) 
386: function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount)
460: function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory)
485: function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable
515: function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner
539: function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner
551: function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner 
563: function setFeeRate(uint16 newFeeRate) public onlyOwner
577: function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner 
588: function setPayRoyalties(bool newPayRoyalties) public onlyOwner
603: function setAllParameters(
        uint128 newVirtualBaseTokenReserves,
        uint128 newVirtualNftReserves,
        bytes32 newMerkleRoot,
        uint16 newFeeRate,
        bool newUseStolenNftOracle,
        bool newPayRoyalties
    ) public {
        setVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);
        setMerkleRoot(newMerkleRoot);
        setFeeRate(newFeeRate);
        setUseStolenNftOracle(newUseStolenNftOracle);
        setPayRoyalties(newPayRoyalties);
    }
775: function flashFeeToken() public view returns (address)

# Use calldata instead of memory
If the suggestion above to change the visibility of functions from public to external is implemented there can be even more gas saved by using calldata instead of memory for some of the functions.
When a function with a memory array is called externally, the abi.decode() step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution. Structs have the same overhead as an array of length one.
Instances include:

386:  function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount)
460: function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory)




# x += y costs more gas than x = x + y for state variables
x += y costs more than x = x + y
same as x -= y
There are 4 instanses of this issue:

### File: src/PrivatePool.sol

231:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
232:  virtualNftReserves -= uint128(weightSum);
324: virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
325: virtualNftReserves += uint128(weightSum);

# Calculating royalties outside of the loop
One pool is only handling one NFT collection and the royalties of the collection are the same for all NFTs. Therefore it will saves gas if the royalties to be paid are calculated outside of the for loop. 
Since the recipient of the royalties is always the same it might also be more gas efficient to send the cumulated royalties in on transaction instead of sending the royalty for each NFT separately since the royalties are calculated from an average price anyway.
### File: src/PrivatePool.sol
212: In the function buy():
Move the following out of the loop:
 if (payRoyalties) {
                // get the royalty fee for the NFT
                (uint256 royaltyFee,) = _getRoyalty(tokenIds[i], salePrice);

                // add the royalty fee to the total royalty fee amount
                royaltyFeeAmount += royaltyFee;
            }

To get the total amount of royalties, multiply the royaltyFee with tokenId.length

302: In the function sell():
Move the following out of the loop:
 if (payRoyalties) {
                // calculate the sale price (assume it's the same for each NFT even if weights differ)
                uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;

                // get the royalty fee for the NFT
                (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

                // tally the royalty fee amount
                royaltyFeeAmount += royaltyFee;

To get the total amount of royalties, multiply the royaltyFee with tokenId.length


# Calculating the salePrice outside of the loop
In the function sell() calculate the salePrice outside of the for loop and use the variable in the for loop instead of calculating it every time the for loop is cycled through.  

### File: src/PrivatePool.sol
336: move the following out of the for loop:
                uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;

# Move Check if Eth amount is right to the top to save gas in case of failure
To fail early move the check if the caller sent 0 ETH if the base token is not ETH to the top of buy() to save gas in case of a revert
### File: src/PrivatePool.sol
226:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
