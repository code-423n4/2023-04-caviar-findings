## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [N-1](#N-1) | Missing checks for `address(0)` when assigning values to address state variables  | 2 |
| [N-2](#N-2) | Use of block.timestamp | 4 |
| [N-3](#N-3) | Maximum Line Length | 103 |
| [N-4](#N-4) |  `require()` / `revert()` statements should have descriptive reason strings  | 1 |
| [N-5](#N-5) | Event is missing `indexed` fields | 12 |
| [N-6](#N-6) | Functions not used internally could be marked external  | 22 |
### [N-1] Missing checks for `address(0)` when assigning values to address state variables 

*Find (2) instance(s) in contracts*:
```solidity
File: Factory.sol

130:         privatePoolMetadata = _privatePoolMetadata;

136:         privatePoolImplementation = _privatePoolImplementation;

```
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)

### [N-2] Use of block.timestamp
Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
#### Recommended Mitigation Steps
Block timestamps should not be used for entropy or generating random numbers — i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.

Instances where block.timestamp is used:

*Find (4) instance(s) in contracts*:
```solidity
File: EthRouter.sol

101:         if (block.timestamp > deadline && deadline != 0) {

154:         if (block.timestamp > deadline && deadline != 0) {

228:         if (block.timestamp > deadline && deadline != 0) {

256:         if (block.timestamp > deadline && deadline != 0) {

```
[EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

### [N-3] Maximum Line Length

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

Reference:
[https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length)

*Find (103) instance(s) in contracts*:
```solidity
File: EthRouter.sol

99:     function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {

109:                 uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(

136:                 ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);

152:     function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public {

162:                 ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);

186:                             getRoyalty(sells[i].nft, sells[i].tokenIds[j], salePrice);

197:                     sells[i].tokenIds, sells[i].tokenWeights, sells[i].proof, sells[i].stolenNftProofs

240:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

254:     function change(Change[] calldata changes, uint256 deadline) public payable {

266:                 ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);

285:                 ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);

307:         address lookupAddress = IRoyaltyRegistry(royaltyRegistry).getRoyaltyLookupAddress(nft);

309:         if (IERC2981(lookupAddress).supportsInterface(type(IERC2981).interfaceId)) {

311:             (recipient, royaltyFee) = IERC2981(lookupAddress).royaltyInfo(tokenId, salePrice);

```
[EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

```solidity
File: Factory.sol

41:     event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);

87:         if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

92:         privatePool = PrivatePool(payable(privatePoolImplementation.cloneDeterministic(_salt)));

115:             ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);

120:             ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

168:     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

169:         predictedAddress = privatePoolImplementation.predictDeterministicAddress(salt, address(this));

```
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)

```solidity
File: PrivatePool.sol

58:     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

59:     event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);

60:     event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);

62:     event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);

63:     event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);

64:     event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);

143:     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {

211:     function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)

214:         returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

219:         uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

236:         uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;

240:             ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

256:             ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);

259:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

265:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

268:             if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);

274:                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

288:         emit Buy(tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);

306:     ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {

310:         uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

317:             IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);

323:         virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

331:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

335:                 uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;

338:                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

362:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

368:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

372:         emit Sell(tokenIds, tokenWeights, netOutputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);

401:             IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, inputTokenIds, stolenNftProofs);

407:             uint256 inputWeightSum = sumWeightsAndValidateProof(inputTokenIds, inputTokenWeights, inputProof);

410:             uint256 outputWeightSum = sumWeightsAndValidateProof(outputTokenIds, outputTokenWeights, outputProof);

413:             if (inputWeightSum < outputWeightSum) revert InsufficientInputWeight();

423:             ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);

426:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);

429:             if (msg.value < feeAmount + protocolFeeAmount) revert InvalidEthAmount();

432:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

436:                 msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);

442:             ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);

447:             ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);

451:         emit Change(inputTokenIds, inputTokenWeights, outputTokenIds, outputTokenWeights, feeAmount, protocolFeeAmount);

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

461:         (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

484:     function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {

489:         if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

497:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

502:             ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);

514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

519:             ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

544:         emit SetVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);

576:     function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

623:     function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data)

629:         if (!availableForFlashLoan(token, tokenId)) revert NotAvailableForFlashLoan();

635:         if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

638:         ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);

642:             receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");

648:         ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);

651:         if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

675:             leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

682:         if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)) {

697:         returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

701:             FixedPointMathLib.mulDivUp(outputAmount, virtualBaseTokenReserves, (virtualNftReserves - outputAmount));

703:         protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() / 10_000;

716:         returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

719:         uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);

721:         protocolFeeAmount = outputAmount * Factory(factory).protocolFeeRate() / 10_000;

731:     function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {

733:         uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;

737:         protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;

744:         uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());

763:     function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {

784:         address lookupAddress = IRoyaltyRegistry(royaltyRegistry).getRoyaltyLookupAddress(nft);

786:         if (IERC2981(lookupAddress).supportsInterface(type(IERC2981).interfaceId)) {

788:             (recipient, royaltyFee) = IERC2981(lookupAddress).royaltyInfo(tokenId, salePrice);

```
[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

```solidity
File: PrivatePoolMetadata.sol

30:         return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));

36:         PrivatePool privatePool = PrivatePool(payable(address(uint160(tokenId))));

56:         PrivatePool privatePool = PrivatePool(payable(address(uint160(tokenId))));

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

```
[PrivatePoolMetadata.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol)

```solidity
File: interfaces/IStolenNftOracle.sol

20:     function validateTokensAreNotStolen(address tokenAddress, uint256[] calldata tokenIds, Message[] calldata proofs)

```
[interfaces/IStolenNftOracle.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol)

### [N-4]  `require()` / `revert()` statements should have descriptive reason strings 

*Find (1) instance(s) in contracts*:
```solidity
File: Factory.sol

88:             revert PrivatePool.InvalidEthAmount();

```
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)


### [N-5] Event is missing `indexed` fields 
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Find (12) instance(s) in contracts*:
```solidity
File: Factory.sol

41:     event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);

```
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)

```solidity
File: PrivatePool.sol

58:     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

59:     event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);

60:     event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);

61:     event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);

62:     event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);

63:     event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);

64:     event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);

65:     event SetMerkleRoot(bytes32 merkleRoot);

66:     event SetFeeRate(uint16 feeRate);

67:     event SetUseStolenNftOracle(bool useStolenNftOracle);

68:     event SetPayRoyalties(bool payRoyalties);

```
[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)



### [N-6] Functions not used internally could be marked external 

*Find (22) instance(s) in contracts*:
```solidity
File: EthRouter.sol

99:     function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {

152:     function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public {

219:     function deposit(

254:     function change(Change[] calldata changes, uint256 deadline) public payable {

```
[EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

```solidity
File: Factory.sol

71:     function create(

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:     function withdraw(address token, uint256 amount) public onlyOwner {

161:     function tokenURI(uint256 id) public view override returns (string memory) {

168:     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

```
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)

```solidity
File: PrivatePool.sol

157:     function initialize(

211:     function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)

301:     function sell(

385:     function change(

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

484:     function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {

514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

602:     function setAllParameters(

742:     function price() public view returns (uint256) {

755:     function flashFeeToken() public view returns (address) {

```
[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

```solidity
File: PrivatePoolMetadata.sol

17:     function tokenURI(uint256 tokenId) public view returns (string memory) {

```
[PrivatePoolMetadata.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol)


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | SOLMATE’S SAFETRANSFERLIB DOESN’T CHECK WHETHER THE ERC20 CONTRACT EXISTS | 38 |
| [L-2](#L-2) | Use safeTransferOwnership instead of transferOwnership function | 2 |
| [L-3](#L-3) | Non-library/interface files should use fixed compiler versions, not floating ones  | 5 |
| [L-4](#L-4) | Unused/Empty RECEIVE()/FALLBACK() function | 3 |
| [L-5](#L-5) | Lack of checks on the values passed to initialize() | 1 |
| [L-6](#L-6) | Should throw an error if _tokenId is not a valid NFT | 1 |
| [L-7](#L-7) | initialize() function can be called by anybody | 1 |


### [L-1] SOLMATE’S SAFETRANSFERLIB DOESN’T CHECK WHETHER THE ERC20 CONTRACT EXISTS
Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).

##### This is stated in the Solmate library: 
https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

*Find (38) instance(s) in contracts*:
```solidity
File: EthRouter.sol

123:                             royaltyRecipient.safeTransferETH(royaltyFee);

136:                 ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);

142:             msg.sender.safeTransferETH(address(this).balance);

162:                 ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);

190:                             royaltyRecipient.safeTransferETH(royaltyFee);

208:         msg.sender.safeTransferETH(address(this).balance);

240:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

266:                 ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);

285:                 ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);

291:             msg.sender.safeTransferETH(address(this).balance);

```
[EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

```solidity
File: Factory.sol

112:             address(privatePool).safeTransferETH(baseTokenAmount);

120:             ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);

150:             msg.sender.safeTransferETH(amount);

```
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)

```solidity
File: PrivatePool.sol

240:             ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

256:             ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);

259:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

265:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

268:             if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);

279:                         ERC20(baseToken).safeTransfer(recipient, royaltyFee);

281:                         recipient.safeTransferETH(royaltyFee);

331:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

346:                         ERC20(baseToken).safeTransfer(recipient, royaltyFee);

348:                         recipient.safeTransferETH(royaltyFee);

359:             msg.sender.safeTransferETH(netOutputAmount);

362:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

368:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

423:             ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);

426:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);

432:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

436:                 msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);

442:             ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);

447:             ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);

497:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

502:             ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);

519:             ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

524:             msg.sender.safeTransferETH(tokenAmount);

638:         ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);

648:         ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);

```
[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

### [L-2] Use safeTransferOwnership instead of transferOwnership function
transferOwnership function is used to change Ownership from Owned.sol.
safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.
    
#### Recommendation:
Use a 2 structure transferOwnership which is safer.
Use [Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) in contracts.

*Find (2) instance(s) in contracts*:
```solidity
File: Factory.sol

26: import {Owned} from "solmate/auth/Owned.sol";

37: contract Factory is ERC721, Owned {

```
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)

### [L-3] Non-library/interface files should use fixed compiler versions, not floating ones

*Find (5) instance(s) in contracts*:
```solidity
File: EthRouter.sol

2: pragma solidity ^0.8.19;

```
[EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

```solidity
File: Factory.sol

2: pragma solidity ^0.8.19;

```
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)

```solidity
File: PrivatePool.sol

2: pragma solidity ^0.8.19;

```
[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

```solidity
File: PrivatePoolMetadata.sol

2: pragma solidity ^0.8.19;

```
[PrivatePoolMetadata.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol)

```solidity
File: interfaces/IStolenNftOracle.sol

2: pragma solidity ^0.8.19;

```
[interfaces/IStolenNftOracle.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol)

### [L-4] Unused/Empty RECEIVE()/FALLBACK() function
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds.

*Find (3) instance(s) in contracts*:
```solidity
File: EthRouter.sol

88:     receive() external payable {}

```
[EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

```solidity
File: Factory.sol

55:     receive() external payable {}

```
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)

```solidity
File: PrivatePool.sol

134:     receive() external payable {}

```
[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

### [L-5]Lack of checks on the values passed to initialize()
Failure to validate input parameters during initialize() may result in errors in the contract after initialization.
similar finding:
https://code4rena.com/reports/2022-10-juicebox/#m-01-multiples-initializations-of-jbtiered721delegate

```solidity
File: PrivatePool.sol.sol

157:      function initialize(
158:        address _baseToken,
159:        address _nft,
160:        uint128 _virtualBaseTokenReserves,
161:        uint128 _virtualNftReserves,
162:        uint56 _changeFee,
163:        uint16 _feeRate,
164:        bytes32 _merkleRoot,
165:       bool _useStolenNftOracle,
166:       bool _payRoyalties
167:   ) public {

```
[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)
#### Recommendation:
Add checks for the values passed to initialize().




### [L-6] Should throw an error if _tokenId is not a valid NFT
According to EIP-721 and specifically, the metadata extension, the tokenURI function should throw an error if _tokenId is not a valid NFT. 

similar finding:
https://code4rena.com/reports/2022-10-juicebox/#l-01-jbtiered721delegatetokenuri-should-throw-an-error-if-_tokenid-is-not-a-valid-nft

```solidity
File: PrivatePoolMetadata.sol

17:      function tokenURI(uint256 tokenId) public view returns (string memory) {
```
[PrivatePoolMetadata.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol)

### [L-7] initialize() function can be called by anybody
In PrivatePool.sol, the comment states that "Initializes the private pool and sets the initial parameters. Should only be called once by the factory." However, the initialize() function is declared as public. If someone calls initialize() before the factory contract does, it will cause the transaction to fail because initialize() has already been initialized.

similar findings:
- https://github.com/code-423n4/2022-03-joyn-findings/issues/5
- https://code4rena.com/reports/2022-11-redactedcartel/#l-02-initialize-function-can-be-called-by-anybody
```solidity
File: PrivatePool.sol.sol

167:      ) public {
168:        // prevent duplicate initialization
169:         if (initialized) revert AlreadyInitialized();
    
185:  // mark the pool as initialized
186:         initialized = true;


```
[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)
#### Recommendation:
Ensure that only the factory contract can call initialize()