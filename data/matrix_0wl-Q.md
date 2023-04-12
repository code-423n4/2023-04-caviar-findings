## Non Critical Issues

|       | Issue                                                                   |
| ----- | :---------------------------------------------------------------------- |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                    |
| NC-2  | ADD TO BLACKLIST FUNCTION                                               |
| NC-3  | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)                    |
| NC-4  | GENERATE PERFECT CODE HEADERS EVERY TIME                                |
| NC-5  | MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL                  |
| NC-6  | MISSING EVENT FOR CRITICAL PARAMETER CHANGE                             |
| NC-7  | MISSING FEE PARAMETER VALIDATION                                        |
| NC-8  | NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS                       |
| NC-9  | INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS                           |
| NC-10 | NO SAME VALUE INPUT CONTROL                                             |
| NC-11 | OMISSIONS IN EVENTS                                                     |
| NC-12 | SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC                      |
| NC-13 | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE     |
| NC-14 | NO CONFIG.SOL FILE                                                      |
| NC-15 | DON’T USE PERIODS WITH FRAGMENTS                                        |
| NC-16 | PRAGMA VERSION^0.8.19 VERSION TOO RECENT TO BE TRUSTED                  |
| NC-17 | UNUSED IMPORTS                                                          |
| NC-18 | FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS |
| NC-19 | THE TOKENADDRESS STATE VARIABLE SHOULD BE RENAMED TO TOKEN              |
| NC-20 | LINES ARE TOO LONG                                                      |
| NC-21 | USE OZ MERKLETREE IMPLEMENTATION INSTEAD OF CREATING A NEW ONE          |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

166:             ERC721(sells[i].nft).setApprovalForAll(sells[i].pool, true);

244:         ERC721(nft).setApprovalForAll(privatePool, true);

270:             ERC721(_change.nft).setApprovalForAll(_change.pool, true);

```

```solidity
File: src/Factory.sol

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

```

```solidity
File: src/PrivatePool.sol

538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550:     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {

576:     function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587:     function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

602:     function setAllParameters(

```

### [NC-2] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

### [NC-3] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED(,)

#### Description:

Rather than using `abi.encodePacked` for appending bytes, since version 0.8.4, `bytes.concat()` is enabled.

Since version 0.8.4 for appending bytes, `bytes.concat()` can be used instead of `abi.encodePacked(,)`.

#### **Proof Of Concept**

```solidity
File: src/PrivatePoolMetadata.sol

30:         return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));

```

### [NC-4] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Context:

All contracts

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-5] MARK VISIBILITY OF INITIALIZE(…) FUNCTIONS AS EXTERNAL

#### Description:

If someone wants to extend via inheritance, it might make more sense that the overridden initialize(...) function calls the internal {...}\_init function, not the parent public initialize(...) function.

External instead of public would give more sense of the initialize(...) functions to behave like a constructor (only called on deployment, so should only be called externally).

From a security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

157:     function initialize(

```

### [NC-6] MISSING EVENT FOR CRITICAL PARAMETER CHANGE

#### Description:

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

166:             ERC721(sells[i].nft).setApprovalForAll(sells[i].pool, true);

244:         ERC721(nft).setApprovalForAll(privatePool, true);

270:             ERC721(_change.nft).setApprovalForAll(_change.pool, true);

```

```solidity
File: src/Factory.sol

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

```

#### Recommendation:

Add Event-Emit

### [NC-7] MISSING FEE PARAMETER VALIDATION

#### Description:

Some fee parameters of functions are not checked for invalid values. Validate the parameters.

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

51:     uint16 public protocolFeeRate;

76:         uint56 _changeFee,

77:         uint16 _feeRate,

142:         protocolFeeRate = _protocolFeeRate;

```

```solidity
File: src/PrivatePool.sol

88:     uint56 public changeFee;

91:     uint16 public feeRate;

```

### [NC-8] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

#### Description:

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.

In complex projects the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html.

NatSpec is missing for the following functions, constructor and modifier:

### [NC-9] INCLUDE RETURN PARAMETERS IN NATSPEC COMMENTS

#### Description:

If Return parameters are declared, you must prefix them with ”/// @return”.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

#### **Proof Of Concept**

```solidity
File: src/PrivatePoolMetadata.sol

30:         return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));

50:         return string(_attributes);

109:         return _svg;

114:         return string(

```

#### Recommended Mitigation Steps:

Include return parameters in NatSpec comments

### [NC-10] NO SAME VALUE INPUT CONTROL

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

91:         royaltyRegistry = _royaltyRegistry;

```

```solidity
File: src/Factory.sol

130:         privatePoolMetadata = _privatePoolMetadata;

136:         privatePoolImplementation = _privatePoolImplementation;

142:         protocolFeeRate = _protocolFeeRate;

```

```solidity
File: src/PrivatePool.sol

145:         royaltyRegistry = _royaltyRegistry;

146:         stolenNftOracle = _stolenNftOracle;

175:         baseToken = _baseToken;

176:         nft = _nft;

177:         virtualBaseTokenReserves = _virtualBaseTokenReserves;

178:         virtualNftReserves = _virtualNftReserves;

179:         changeFee = _changeFee;

180:         feeRate = _feeRate;

181:         merkleRoot = _merkleRoot;

182:         useStolenNftOracle = _useStolenNftOracle;

183:         payRoyalties = _payRoyalties;

```

### [NC-11] OMISSIONS IN EVENTS

#### Description:

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters.

The events should include the new value and old value where possible.

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

555:         emit SetMerkleRoot(newMerkleRoot);

570:         emit SetFeeRate(newFeeRate);

581:         emit SetUseStolenNftOracle(newUseStolenNftOracle);

592:         emit SetPayRoyalties(newPayRoyalties);

```

### [NC-12] SOLIDITY COMPILER OPTIMIZATIONS CAN BE PROBLEMATIC

#### Description:

Protocol has enabled optional compiler optimizations in Solidity. There have been several optimization bugs with security implications. Moreover, optimizations are actively being developed. Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them.

Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten-generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG.

Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. More recently, another bug due to the incorrect caching of keccak256 was reported. A compiler audit of Solidity from November 2018 concluded that the optional optimizations may not be safe. It is likely that there are latent bugs related to optimization and that new bugs will be introduced due to future optimizations.

#### Exploit Scenario:

A latent or future bug in Solidity compiler optimizations—or in the Emscripten transpilation to solc-js—causes a security vulnerability in the contracts.

#### **Proof Of Concept**

```solidity
File: foundry.toml

optimizer_runs = 200
```

### [NC-13] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

#### Context:

src/PrivatePool.sol

#### Description:

Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private, within a grouping, place the view and pure functions last

### [NC-14] NO CONFIG.SOL FILE

Better to have config facet, in case some update is needed in the config.sol.

### [NC-15] DON’T USE PERIODS WITH FRAGMENTS

#### Context:

All Contracts

#### Description:

Throughout the files, some of the comments have fragments that end with periods and some of them end without periods. It should be consistent. They should either be converted to actual sentences with both a noun phrase and a verb phrase, or the periods should be removed.

### [NC-16] PRAGMA VERSION^0.8.19 VERSION TOO RECENT TO BE TRUSTED

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/Factory.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/PrivatePool.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/PrivatePoolMetadata.sol

2: pragma solidity ^0.8.19;

```

```solidity
File: src/interfaces/IStolenNftOracle.sol

2: pragma solidity ^0.8.19;

```

### [NC-17] UNUSED IMPORTS

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

29: import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";

31: import {MerkleProofLib} from "solady/utils/MerkleProofLib.sol";

32: import {IERC2981} from "openzeppelin/interfaces/IERC2981.sol";

33: import {IRoyaltyRegistry} from "royalty-registry-solidity/IRoyaltyRegistry.sol";

34: import {IERC3156FlashBorrower} from "openzeppelin/interfaces/IERC3156FlashLender.sol";

37: import {Factory} from "./Factory.sol";

```

### [NC-18] FOR FUNCTIONS AND VARIABLES FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS

#### Description:

Solidity’s standard naming convention for internal and private functions and variables (apart from constants): the mixedCase format starting with an underscore (\_mixedCase starting with an underscore)Solidity’s standard naming convention for internal and private constants variables: the snake_case format starting with an underscore (\_mixedCase starting with an underscore) and use ALL_CAPS for naming them.

#### **Proof Of Concept**

```solidity
File: src/PrivatePoolMetadata.sol


112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

```

### [NC-19] THE TOKENADDRESS STATE VARIABLE SHOULD BE RENAMED TO TOKEN

#### Description:

The tokenAddress state variable of IStolenNftOracle.sol should be renamed to token as this variable represents an IERC20 interface rather that just an address. Renaming it to token aligns better with its usage.

#### **Proof Of Concept**

```solidity
File: src/interfaces/IStolenNftOracle.sol

20:     function validateTokensAreNotStolen(address tokenAddress, uint256[] calldata tokenIds, Message[] calldata proofs)

```

### [NC-20] LINES ARE TOO LONG

#### Description:

Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

35: import {IRoyaltyRegistry} from "royalty-registry-solidity/IRoyaltyRegistry.sol";

99:     function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {

109:                 uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(

119:                             getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);

136:                 ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);

152:     function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public {

162:                 ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);

177:                     abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))

182:                     uint256 salePrice = outputAmount / sells[i].tokenIds.length;

186:                             getRoyalty(sells[i].nft, sells[i].tokenIds[j], salePrice);

197:                     sells[i].tokenIds, sells[i].tokenWeights, sells[i].proof, sells[i].stolenNftProofs

240:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

247:         PrivatePool(privatePool).deposit{value: msg.value}(tokenIds, msg.value);

254:     function change(Change[] calldata changes, uint256 deadline) public payable {

266:                 ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);

285:                 ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);

307:         address lookupAddress = IRoyaltyRegistry(royaltyRegistry).getRoyaltyLookupAddress(nft);

309:         if (IERC2981(lookupAddress).supportsInterface(type(IERC2981).interfaceId)) {

311:             (recipient, royaltyFee) = IERC2981(lookupAddress).royaltyInfo(tokenId, salePrice);

```

```solidity
File: src/Factory.sol

41:     event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);

82:         uint256[] memory tokenIds, // put in memory to avoid stack too deep error

87:         if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

92:         privatePool = PrivatePool(payable(privatePoolImplementation.cloneDeterministic(_salt)));

115:             ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);

120:             ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

161:     function tokenURI(uint256 id) public view override returns (string memory) {

168:     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

169:         predictedAddress = privatePoolImplementation.predictDeterministicAddress(salt, address(this));

```

```solidity
File: src/PrivatePool.sol

33: import {IRoyaltyRegistry} from "royalty-registry-solidity/IRoyaltyRegistry.sol";

34: import {IERC3156FlashBorrower} from "openzeppelin/interfaces/IERC3156FlashLender.sol";

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

225:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

236:         uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;

240:             ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

256:             ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);

259:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

265:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

268:             if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);

274:                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

288:         emit Buy(tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);

305:         IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error

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

397:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

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

745:         return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;

763:     function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {

784:         address lookupAddress = IRoyaltyRegistry(royaltyRegistry).getRoyaltyLookupAddress(nft);

786:         if (IERC2981(lookupAddress).supportsInterface(type(IERC2981).interfaceId)) {

788:             (recipient, royaltyFee) = IERC2981(lookupAddress).royaltyInfo(tokenId, salePrice);

```

```solidity
File: src/PrivatePoolMetadata.sol

23:                 '"image": ','"data:image/svg+xml;base64,', Base64.encode(svg(tokenId)),'",',

30:         return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));

36:         PrivatePool privatePool = PrivatePool(payable(address(uint160(tokenId))));

40:             trait("Pool address", Strings.toHexString(address(privatePool))), ',',

41:             trait("Base token", Strings.toHexString(privatePool.baseToken())), ',',

43:             trait("Virtual base token reserves",Strings.toString(privatePool.virtualBaseTokenReserves())), ',',

44:             trait("Virtual NFT reserves", Strings.toString(privatePool.virtualNftReserves())), ',',

45:             trait("Fee rate (bps): ", Strings.toString(privatePool.feeRate())), ',',

46:             trait("NFT balance", Strings.toString(ERC721(privatePool.nft()).balanceOf(address(privatePool)))), ',',

47:             trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))

56:         PrivatePool privatePool = PrivatePool(payable(address(uint160(tokenId))));

63:                 '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 400 400" style="width:100%;background:black;fill:white;font-family:serif;">',

68:                         "Private pool: ", Strings.toHexString(address(privatePool)),

71:                         "Base token: ", Strings.toHexString(privatePool.baseToken()),

84:                     "Virtual base token reserves: ", Strings.toString(privatePool.virtualBaseTokenReserves()),

87:                     "Virtual NFT reserves: ", Strings.toString(privatePool.virtualNftReserves()),

90:                     "Fee rate (bps): ", Strings.toString(privatePool.feeRate()),

100:                         "NFT balance: ", Strings.toString(ERC721(privatePool.nft()).balanceOf(address(privatePool))),

103:                         "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

116:                 '{ "trait_type": "', traitType, '",', '"value": "', value, '" }'

```

```solidity
File: src/interfaces/IStolenNftOracle.sol

20:     function validateTokensAreNotStolen(address tokenAddress, uint256[] calldata tokenIds, Message[] calldata proofs)

```

### [NC-21] USE OZ MERKLETREE IMPLEMENTATION INSTEAD OF CREATING A NEW ONE

#### Description:

Instead of your own merkle tree lib. Use openzeppelin implementation;

https://docs.openzeppelin.com/contracts/4.x/api/utils#MerkleProof

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

53:         PrivatePool.MerkleMultiProof proof;

63:         PrivatePool.MerkleMultiProof proof;

74:         PrivatePool.MerkleMultiProof inputProof;

78:         PrivatePool.MerkleMultiProof outputProof;

```

```solidity
File: src/Factory.sol

78:         bytes32 _merkleRoot,

105:             _merkleRoot,

```

```solidity
File: src/PrivatePool.sol

31: import {MerkleProofLib} from "solady/utils/MerkleProofLib.sol";

31: import {MerkleProofLib} from "solady/utils/MerkleProofLib.sol";

52:     struct MerkleMultiProof {

58:     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

65:     event SetMerkleRoot(bytes32 merkleRoot);

65:     event SetMerkleRoot(bytes32 merkleRoot);

74:     error InvalidMerkleProof();

116:     bytes32 public merkleRoot;

164:         bytes32 _merkleRoot,

181:         merkleRoot = _merkleRoot;

181:         merkleRoot = _merkleRoot;

196:             _merkleRoot,

211:     function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)

304:         MerkleMultiProof calldata proof,

388:         MerkleMultiProof memory inputProof,

392:         MerkleMultiProof memory outputProof

550:     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

550:     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

552:         merkleRoot = newMerkleRoot;

552:         merkleRoot = newMerkleRoot;

555:         emit SetMerkleRoot(newMerkleRoot);

555:         emit SetMerkleRoot(newMerkleRoot);

605:         bytes32 newMerkleRoot,

611:         setMerkleRoot(newMerkleRoot);

611:         setMerkleRoot(newMerkleRoot);

664:         MerkleMultiProof memory proof

667:         if (merkleRoot == bytes32(0)) {

682:         if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)) {

682:         if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)) {

683:             revert InvalidMerkleProof();

```

## Low Issues

|      | Issue                                                                          |
| ---- | :----------------------------------------------------------------------------- |
| L-1  | ARBITRARY JUMP WITH FUNCTION TYPE VARIABLE                                     |
| L-2  | INITIALIZING STATE-VARIABLES IN PROXY-BASED UPGRADEABLE CONTRACTS              |
| L-3  | DOS WITH BLOCK GAS LIMIT                                                       |
| L-4  | AVOID `TRANSFER()`/`SEND()` AS REENTRANCY MITIGATIONS                          |
| L-5  | CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE                         |
| L-6  | DOS WITH FAILED CALL                                                           |
| L-7  | LACK OF INPUT VALIDATION                                                       |
| L-8  | LOSS OF PRECISION DUE TO ROUNDING                                              |
| L-9  | UNIFY RETURN CRITERIA                                                          |
| L-10 | INCORRECT RETURN VALUES FOR ERC721 `OWNEROF()`                                 |
| L-11 | REVERT MESSAGE IS TOO SHORT AND UNCLEAR                                        |
| L-12 | THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS |
| L-13 | SOLMATE’S SAFETRANSFERLIB DOESN’T CHECK WHETHER THE ERC20 CONTRACT EXISTS      |
| L-14 | MODIFIER SIDE-EFFECTS                                                          |
| L-15 | A SINGLE POINT OF FAILURE                                                      |
| L-16 | BLOCK VALUES AS A PROXY FOR TIME                                               |
| L-17 | ACCOUNT EXISTENCE CHECK FOR LOW-LEVEL CALLS                                    |
| L-18 | UNSAFE CAST                                                                    |
| L-19 | UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT                        |
| L-20 | USE `SAFETRANSFER` INSTEAD OF `TRANSFER`                                       |
| L-21 | USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION            |

### [L-1] ARBITRARY JUMP WITH FUNCTION TYPE VARIABLE

#### Description:

Function types are supported in Solidity. This means that a variable of type function can be assigned to a function with a matching signature. The function can then be called from the variable just like any other function. Users should not be able to change the function variable, but in some cases this is possible.

If the smart contract uses certain assembly instructions, mstore for example, an attacker may be able to point the function variable to any other function. This may give the attacker the ability to break the functionality of the contract, and perhaps even drain the contract funds.

Since inline assembly is a way to access the EVM at a low level, it bypasses many important safety features. So it's important to only use assembly if it is necessary and properly understood.

[SWC Registry](https://swcregistry.io/docs/SWC-127)

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

469:             assembly {
                    let returnData_size := mload(returnData)

```

### [L-2] INITIALIZING STATE-VARIABLES IN PROXY-BASED UPGRADEABLE CONTRACTS

#### Description:

This should be done in initializer functions and not as part of the state variable declarations in which case they won’t be set.

https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#avoid-initial-values-in-field-declarations

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

157:     function initialize(

```

### [L-3] DOS WITH BLOCK GAS LIMIT

#### Description:

Programming patterns such as looping over arrays of unknown size may lead to DoS when the gas cost of execution exceeds the block gas limit.

Reference: https://swcregistry.io/docs/SWC-128

This loops could drain all user gas and revert.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

106:     for (uint256 i = 0; i < buys.length; i++) {

116:     for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

134:     for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

159:     for (uint256 i = 0; i < sells.length; i++) {

161:     for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

183:     for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

239:     for (uint256 i = 0; i < tokenIds.length; i++) {

261:     for (uint256 i = 0; i < changes.length; i++) {

265:     for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {

284:     for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {

```

```solidity
File: src/Factory.sol

119:                 for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: src/PrivatePool.sol

238:     for (uint256 i = 0; i < tokenIds.length; i++) {

272:     for (uint256 i = 0; i < tokenIds.length; i++) {

329:     for (uint256 i = 0; i < tokenIds.length; i++) {

441:     for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:     for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:     for (uint256 i = 0; i < tokenIds.length; i++) {

518:     for (uint256 i = 0; i < tokenIds.length; i++) {

673:     for (uint256 i = 0; i < tokenIds.length; i++) {

```

### [L-4] AVOID `TRANSFER()`/`SEND()` AS REENTRANCY MITIGATIONS

#### Description:

Although `transfer()` and `send()` have been recommended as a security best-practice to prevent reentrancy attacks because they only forward 2300 gas, the gas repricing of opcodes may break deployed contracts. Use `call()` instead, without hardcoded gas limits along with checks-effects-interactions pattern or reentrancy guards for reentrancy protection.

https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/

https://swcregistry.io/docs/SWC-134

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Using low-level `call.value(amount)` with the corresponding result check or using the OpenZeppelin `Address.sendValue` is advised:https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L60

```solidity
File: src/Factory.sol

152:             ERC20(token).transfer(msg.sender, amount);

```

```solidity
File: src/PrivatePool.sol

365:             ERC20(baseToken).transfer(msg.sender, netOutputAmount);

527:             ERC20(token).transfer(msg.sender, tokenAmount);

```

### [L-5] CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE

#### Description:

The critical procedures should be two step process.

Changing critical addresses in contracts should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one (i.e. claims ownership). This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1488) and [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2369))

#### **Proof Of Concept**

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

```solidity
File: src/Factory.sol

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

```

### [L-6] DOS WITH FAILED CALL

#### Description:

External calls can fail accidentally or deliberately, which can cause a DoS condition in the contract. To minimize the damage caused by such failures, it is better to isolate each external call into its own transaction that can be initiated by the recipient of the call. This is especially relevant for payments, where it is better to let users withdraw funds rather than push funds to them automatically (this also reduces the chance of problems with the gas limit).

https://swcregistry.io/docs/SWC-113

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

123:            royaltyRecipient.safeTransferETH(royaltyFee);

136:                 ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);

162:                 ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);

190:            royaltyRecipient.safeTransferETH(royaltyFee);

240:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

266:                 ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);

285:                 ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);

```

```solidity
File: src/Factory.sol

120:             ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);

```

```solidity
File: src/PrivatePool.sol

240:             ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

279:            ERC20(baseToken).safeTransfer(recipient, royaltyFee);

331:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

346:            ERC20(baseToken).safeTransfer(recipient, royaltyFee);

348:            recipient.safeTransferETH(royaltyFee);

442:             ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);

447:             ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);

497:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

519:             ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

524:             msg.sender.safeTransferETH(tokenAmount);

527:             ERC20(token).transfer(msg.sender, tokenAmount);

```

### [L-7] LACK OF INPUT VALIDATION

#### Description:

For defence-in-depth purpose, it is recommended to perform additional validation against the amount that the user is attempting to deposit, mint, withdraw and redeem to ensure that the submitted amount is valid.

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

148:     function withdraw(address token, uint256 amount) public onlyOwner {

```

### [L-8] LOSS OF PRECISION DUE TO ROUNDING

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

703:         protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() / 10_000;

704:         feeAmount = inputAmount * feeRate / 10_000;

719:         uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);

721:         protocolFeeAmount = outputAmount * Factory(factory).protocolFeeRate() / 10_000;

722:         feeAmount = outputAmount * feeRate / 10_000;

736:         feeAmount = inputAmount * feePerNft / 1e18;

737:         protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;

745:         return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;

```

### [L-9] UNIFY RETURN CRITERIA

#### Description:

In EthRouter.sol, Factory.sol, PrivatePool.sol and PrivatePoolMetadata.sol files, the methods are sometimes returned with return, other times the return variable is equal, and on other occasions the name of the return variable is not defined, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

304:         returns (uint256 royaltyFee, address recipient)

```

```solidity
File: src/Factory.sol

84:     ) public payable returns (PrivatePool privatePool) {

161:     function tokenURI(uint256 id) public view override returns (string memory) {

168:     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

```

```solidity
File: src/PrivatePool.sol

214:         returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

306:     ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {

393:     ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

626:         returns (bool)

665:     ) public view returns (uint256) {

697:         returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

716:         returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

731:     function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {

742:     function price() public view returns (uint256) {

750:     function flashFee(address, uint256) public view returns (uint256) {

755:     function flashFeeToken() public view returns (address) {

763:     function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {

765:         try ERC721(token).ownerOf(tokenId) returns (address result) {

781:         returns (uint256 royaltyFee, address recipient)

```

```solidity
File: src/PrivatePoolMetadata.sol

17:     function tokenURI(uint256 tokenId) public view returns (string memory) {

35:     function attributes(uint256 tokenId) public view returns (string memory) {

55:     function svg(uint256 tokenId) public view returns (bytes memory) {

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

```

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

### [L-10] INCORRECT RETURN VALUES FOR ERC721 `OWNEROF()`

#### Description:

Incorrect return values for ERC721 ownerOf(): Contracts compiled with solc >= 0.4.22 interacting with ERC721 ownerOf() that returns a bool instead of address type will revert. Use OpenZeppelin’s ERC721 contracts.

Reference: https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-erc721-interface

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

128:         if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {

765:         try ERC721(token).ownerOf(tokenId) returns (address result) {

```

### [L-11] REVERT MESSAGE IS TOO SHORT AND UNCLEAR

#### Description:

The correct and clear error description explains to the user why the function reverts, but the error descriptions below in the project are not self-explanatory. These error descriptions are very important in the debug features. Error definitions should be added to the revert block, not exceeding 32 bytes.

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

474:             revert();

```

#### Recommended Mitigation Steps:

Error definitions should be added to the revert block, not exceeding 32 bytes or we should use custom errors.

### [L-12] THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS

#### Description:

If a pair gets created and after a while one of the tokens gets self-destroyed (maybe because of a bug) then `safeTransfer` would still succeed. It’s probably a good idea to check if the contract still exists by checking the bytecode length.

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

259:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

279:                         ERC20(baseToken).safeTransfer(recipient, royaltyFee);

346:                         ERC20(baseToken).safeTransfer(recipient, royaltyFee);

368:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

```

### [L-13] SOLMATE’S SAFETRANSFERLIB DOESN’T CHECK WHETHER THE ERC20 CONTRACT EXISTS

#### Description:

Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).

This is stated in the Solmate library: https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

32: import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";

46:     using SafeTransferLib for address;

```

```solidity
File: src/Factory.sol

27: import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";

39:     using SafeTransferLib for address;

```

```solidity
File: src/PrivatePool.sol

30: import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";

46:     using SafeTransferLib for address payable;

47:     using SafeTransferLib for address;

48:     using SafeTransferLib for ERC20;

```

### [L-14] MODIFIER SIDE-EFFECTS

#### Description:

Modifiers should only implement checks and not make state changes and external calls which violates the [checks-effects-interactions](https://solidity.readthedocs.io/en/develop/security-considerations.html#use-the-checks-effects-interactions-pattern) pattern. These side-effects may go unnoticed by developers/auditors because the modifier code is typically far from the function implementation.

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

127:     modifier onlyOwner() virtual {
            if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
            revert Unauthorized();
        }
        _;
    }

```

### [L-15] A SINGLE POINT OF FAILURE

#### Description:

The owner role has a single point of failure and `onlyOwner` can use critical a few functions.

Owner is not behind a multisig and changes are not behind a timelock.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:     function withdraw(address token, uint256 amount) public onlyOwner {

```

```solidity
File: src/PrivatePool.sol

127:     modifier onlyOwner() virtual {

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550:     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {

576:     function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587:     function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```

#### Recommended Mitigation Steps:

Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Allow only multi-signature wallets to call the function to reduce the likelihood of an attack.

https://twitter.com/danielvf/status/1572963475101556738?s=20&t=V1kvzfJlsx-D2hfnG0OmuQ

Also detail them in documentation and NatSpec comments.

### [L-16] BLOCK VALUES AS A PROXY FOR TIME

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

Reference: https://swcregistry.io/docs/SWC-116

Reference: (https://github.com/kadenzipfel/smart-contract-vulnerabilities/blob/master/vulnerabilities/timestamp-dependence.md)

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

101:         if (block.timestamp > deadline && deadline != 0) {

154:         if (block.timestamp > deadline && deadline != 0) {

228:         if (block.timestamp > deadline && deadline != 0) {

256:         if (block.timestamp > deadline && deadline != 0) {

```

### [L-17] ACCOUNT EXISTENCE CHECK FOR LOW-LEVEL CALLS

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

461:         (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

```

#### Recommended Mitigation Steps:

In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

### [L-18] UNSAFE CAST

#### Description:

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231:         virtualNftReserves -= uint128(weightSum);

323:         virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324:         virtualNftReserves += uint128(weightSum);

```

#### Recommended Mitigation Steps:

Use a safeCast from Open Zeppelin or increase the type length.

### [L-19] UNUSED `RECEIVE()` FUNCTION WILL LOCK ETHER IN CONTRACT

#### Description:

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

88:     receive() external payable {}

```

```solidity
File: src/Factory.sol

55:     receive() external payable {}

```

```solidity
File: src/PrivatePool.sol

134:     receive() external payable {}

```

#### Recommended Mitigation Steps:

The function should call another function, otherwise it should revert

### [L-20] USE `SAFETRANSFER` INSTEAD OF `TRANSFER`

#### Description:

It is good to add a `require()` statement that checks the return value of token transfers or to use something like OpenZeppelin’s `safeTransfer`/`safeTransferFrom` unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

For example, Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)‘s `transfer()` and `transferFrom()` functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert.

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

152:             ERC20(token).transfer(msg.sender, amount);

```

```solidity
File: src/PrivatePool.sol

365:             ERC20(baseToken).transfer(msg.sender, netOutputAmount);

527:             ERC20(token).transfer(msg.sender, tokenAmount);

```

#### Recommended Mitigation Steps:

Consider using `safeTransfer`/`safeTransferFrom` or `require()` consistently.

### [L-21] USE `SAFETRANSFEROWNERSHIP` INSTEAD OF `TRANSFEROWNERSHIP` FUNCTION

#### Description:

transferOwnership function is used to change Ownership from Owned.sol.

Use a 2 structure transferOwnership which is safer.

safeTransferOwnership, use it is more secure due to 2-stage ownership transfer.

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

26: import {Owned} from "solmate/auth/Owned.sol";

```

#### Recommended Mitigation Steps:

Use Ownable2Step.sol
