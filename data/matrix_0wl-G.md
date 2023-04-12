## Gas Optimizations

|        | Issue                                                                                                                                               |
| ------ | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | USE `SELFBALANCE()` INSTEAD OF `ADDRESS(THIS).BALANCE`                                                                                              |
| GAS-2  | `<X> += <Y>`/`<X> -= <Y>` COSTS MORE GAS THAN `<X> = <X> + <Y>`/`<X> = <X> - <Y>` FOR STATE VARIABLES                                               |
| GAS-3  | `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`                                                                                          |
| GAS-4  | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                                                                         |
| GAS-5  | SETTING THE CONSTRUCTOR TO PAYABLE                                                                                                                  |
| GAS-6  | DIV BY 0                                                                                                                                            |
| GAS-7  | DOS WITH BLOCK GAS LIMIT                                                                                                                            |
| GAS-8  | USE FUNCTION INSTEAD OF MODIFIERS                                                                                                                   |
| GAS-9  | INSTEAD OF CALCULATING A STATEVAR WITH KECCAK256() EVERY TIME THE CONTRACT IS MADE PRE CALCULATE THEM BEFORE AND ONLY GIVE THE RESULT TO A CONSTANT |
| GAS-10 | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT                                                            |
| GAS-11 | KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE                                                                         |
| GAS-12 | MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL                                                                                        |
| GAS-13 | FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE                                                                    |
| GAS-14 | OPTIMIZE NAMES TO SAVE GAS                                                                                                                          |
| GAS-15 | OPTIMIZE NFT DELEGATE DEPLOYMENTS BY USING PROXY                                                                                                    |
| GAS-16 | THE INCREMENT IN FOR LOOP POSTCONDITION CAN BE MADE UNCHECKED                                                                                       |
| GAS-17 | PROPER DATA TYPES                                                                                                                                   |
| GAS-18 | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD OR FUNCTIONS NOT USED INTERNALLY COULD BE MARKED EXTERNAL           |
| GAS-19 | REORDER STRUCTURE LAYOUT                                                                                                                            |
| GAS-20 | USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS                                                                                        |
| GAS-21 | STRUCTS CAN BE PACKED INTO FEWER STORAGE SLOTS                                                                                                      |
| GAS-22 | TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT                                                                                                 |
| GAS-23 | USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD                                                                              |
| GAS-24 | USE BYTES32 INSTEAD OF STRING                                                                                                                       |

### [GAS-1] USE `SELFBALANCE()` INSTEAD OF `ADDRESS(THIS).BALANCE`

#### Description:

Use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

141:         if (address(this).balance > 0) {

142:             msg.sender.safeTransferETH(address(this).balance);

203:         if (address(this).balance < minOutputAmount) {

208:         msg.sender.safeTransferETH(address(this).balance);

290:         if (address(this).balance > 0) {

291:             msg.sender.safeTransferETH(address(this).balance);

```

### [GAS-2] `<X> += <Y>`/`<X> -= <Y>` COSTS MORE GAS THAN `<X> = <X> + <Y>`/`<X> = <X> - <Y>` FOR STATE VARIABLES

#### Description:

Using the addition operator instead of plus-equals saves gas

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231:         virtualNftReserves -= uint128(weightSum);

247:                 royaltyFeeAmount += royaltyFee;

252:         netInputAmount += royaltyFeeAmount;

323:         virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324:         virtualNftReserves += uint128(weightSum);

341:                 royaltyFeeAmount += royaltyFee;

355:         netOutputAmount -= royaltyFeeAmount;

678:             sum += tokenWeights[i];

```

### [GAS-3] `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`

#### Description:

Use `abi.encodePacked()` where possible to save gas.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

177:                     abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))

```

```solidity
File: src/PrivatePool.sol

675:             leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

```

### [GAS-4] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

#### Description:

Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything:

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

115:             ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);

152:             ERC20(token).transfer(msg.sender, amount);

```

```solidity
File: src/PrivatePool.sol

365:             ERC20(baseToken).transfer(msg.sender, netOutputAmount);

527:             ERC20(token).transfer(msg.sender, tokenAmount);

651:         if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

```

### [GAS-5] SETTING THE CONSTRUCTOR TO PAYABLE

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

90:     constructor(address _royaltyRegistry) {

```

```solidity
File: src/Factory.sol

53:     constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}

```

```solidity
File: src/PrivatePool.sol

143:     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {

```

### [GAS-6] DIV BY 0

#### Description:

Division by 0 can lead to accidentally revert, (An example of a similar issue - https://github.com/code-423n4/2021-10-defiprotocol-findings/issues/84).

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

115:                     uint256 salePrice = inputAmount / buys[i].tokenIds.length;

182:                     uint256 salePrice = outputAmount / sells[i].tokenIds.length;

```

```solidity
File: src/PrivatePool.sol

236:         uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;

335:                 uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;

719:         uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);

745:         return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;

```

### [GAS-7] DOS WITH BLOCK GAS LIMIT

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

#### Recommended Mitigation Steps:

Caution is advised when you expect to have large arrays that grow over time. Actions that require looping across the entire data structure should be avoided.

If you absolutely must loop over an array of unknown size, then you should plan for it to potentially take multiple blocks, and therefore require multiple transactions.

### [GAS-8] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

127:     modifier onlyOwner() virtual {

```

### [GAS-9] INSTEAD OF CALCULATING A STATEVAR WITH KECCAK256() EVERY TIME THE CONTRACT IS MADE PRE CALCULATE THEM BEFORE AND ONLY GIVE THE RESULT TO A CONSTANT

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

642:             receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");

```

### [GAS-10] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

642:             receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");

```

### [GAS-11] KECCAK256() SHOULD ONLY NEED TO BE CALLED ON A SPECIFIC STRING LITERAL ONCE

#### Description:

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

642:             receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");

```

### [GAS-12] MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL

#### **Proof Of Concept**

```solidity
File: src/PrivatePool.sol

127:     modifier onlyOwner() virtual {

```

### [GAS-13] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

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

### [GAS-14] OPTIMIZE NAMES TO SAVE GAS

#### Description:

`public`/`external` function names and `public` member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. In this report are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).

[Solidity optimize name github](https://github.com/enzosv/solidity-optimize-name)

[Source](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

45: contract EthRouter is ERC721TokenReceiver {

```

```solidity
File: src/Factory.sol

37: contract Factory is ERC721, Owned {

```

```solidity
File: src/PrivatePool.sol

45: contract PrivatePool is ERC721TokenReceiver {

```

```solidity
File: src/PrivatePoolMetadata.sol

14: contract PrivatePoolMetadata {

```

```solidity
File: src/interfaces/IStolenNftOracle.sol

4: interface IStolenNftOracle {

```

### [GAS-15] OPTIMIZE NFT DELEGATE DEPLOYMENTS BY USING PROXY

#### Description:

The cost of NFT delegate deployments can be significantly reduced by deploying proxies instead of clones of the implementation.

It copies the code of an existing contract and deploys a new contract with the same code. This is a costly operation because each of the three contracts is a big contract with a lot of code. It’ll be much cheaper to deploy non-upgradable proxies instead.

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

92:         privatePool = PrivatePool(payable(privatePoolImplementation.cloneDeterministic(_salt)));

```

#### Recommended Mitigation Steps:

Consider using the Clones library from OpenZeppelin–it deploys and absolutely minimal non-upgradable proxy contract. Such proxies, however, cannot be verified on Etherscan. Some more info.

### [GAS-16] THE INCREMENT IN FOR LOOP POSTCONDITION CAN BE MADE UNCHECKED

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.

The for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks. One can manually do this.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

106:         for (uint256 i = 0; i < buys.length; i++) {

116:                     for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

134:             for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

159:         for (uint256 i = 0; i < sells.length; i++) {

161:             for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

183:                     for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

239:         for (uint256 i = 0; i < tokenIds.length; i++) {

261:         for (uint256 i = 0; i < changes.length; i++) {

265:             for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {

284:             for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {

```

```solidity
File: src/Factory.sol

119:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: src/PrivatePool.sol

238:         for (uint256 i = 0; i < tokenIds.length; i++) {

272:             for (uint256 i = 0; i < tokenIds.length; i++) {

329:         for (uint256 i = 0; i < tokenIds.length; i++) {

441:         for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:         for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:         for (uint256 i = 0; i < tokenIds.length; i++) {

518:         for (uint256 i = 0; i < tokenIds.length; i++) {

673:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

### [GAS-17] PROPER DATA TYPES

#### Description:

In Solidity, some data types are more expensive than others. It’s important to be aware of the most efficient type that can be used. Here are a few rules about data types.

Type uint should be used in place of type string whenever possible.

Type uint256 takes less gas to store than uint8

Type bytes should be used over byte[]

If the length of bytes can be limited, use the lowest amount possible from bytes1 to bytes32.

Type bytes32 is cheaper to use than type string and bytes.

If data can fit into 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is cheaper in solidity.

[Source](https://betterprogramming.pub/how-to-write-smart-contracts-that-optimize-gas-spent-on-ethereum-30b5e9c5db85)

Fixed size variables are always cheaper than dynamic ones.

[Source](https://medium.com/coinmonks/gas-optimization-in-solidity-part-i-variables-9d5775e43dde)

Most of the time it will be better to use a mapping instead of an array because of its cheaper operations.

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

51:     uint16 public protocolFeeRate;

74:         uint128 _virtualBaseTokenReserves,

75:         uint128 _virtualNftReserves,

76:         uint56 _changeFee,

77:         uint16 _feeRate,

95:         _safeMint(msg.sender, uint256(uint160(address(privatePool))));

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

```

```solidity
File: src/PrivatePool.sol

58:     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

64:     event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);

66:     event SetFeeRate(uint16 feeRate);

88:     uint56 public changeFee;

91:     uint16 public feeRate;

104:     uint128 public virtualBaseTokenReserves;

112:     uint128 public virtualNftReserves;

128:         if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {

160:         uint128 _virtualBaseTokenReserves,

161:         uint128 _virtualNftReserves,

162:         uint56 _changeFee,

163:         uint16 _feeRate,

230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231:         virtualNftReserves -= uint128(weightSum);

323:         virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324:         virtualNftReserves += uint128(weightSum);

538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {

603:         uint128 newVirtualBaseTokenReserves,

604:         uint128 newVirtualNftReserves,

606:         uint16 newFeeRate,

```

```solidity
File: src/PrivatePoolMetadata.sol

56:         PrivatePool privatePool = PrivatePool(payable(address(uint160(tokenId))));

59:         bytes memory _svg;

```

```solidity
File: src/interfaces/IStolenNftOracle.sol

8:         bytes payload;

12:         bytes signature;

```

### [GAS-18] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD OR FUNCTIONS NOT USED INTERNALLY COULD BE MARKED EXTERNAL

#### Description:

The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

99:     function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {

152:     function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public {

254:     function change(Change[] calldata changes, uint256 deadline) public payable {

```

```solidity
File: src/Factory.sol

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:     function withdraw(address token, uint256 amount) public onlyOwner {

161:     function tokenURI(uint256 id) public view override returns (string memory) {

168:     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

```

```solidity
File: src/PrivatePool.sol

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

484:     function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {

742:     function price() public view returns (uint256) {

755:     function flashFeeToken() public view returns (address) {

```

```solidity
File: src/PrivatePoolMetadata.sol

17:     function tokenURI(uint256 tokenId) public view returns (string memory) {

```

### [GAS-19] REORDER STRUCTURE LAYOUT

#### Description:

Structures could be optimized moving the position of certain values in order to save a lot slots.

For example Enums are represented by integers; the possibility listed first by 0, the next by 1, and so forth. An enum type just acts like uintN, where N is the smallest legal value large enough to accomodate all the possibilities.

[Source](https://ethdebug.github.io/solidity-data-representation)

[Source](https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#storage-inplace-encoding)

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

48:     struct Buy {
            address payable pool;
            address nft;
            uint256[] tokenIds;
            uint256[] tokenWeights;
            PrivatePool.MerkleMultiProof proof;
            uint256 baseTokenAmount;
            bool isPublicPool;
        }

58:     struct Sell {
            address payable pool;
            address nft;
            uint256[] tokenIds;
            uint256[] tokenWeights;
            PrivatePool.MerkleMultiProof proof;
            IStolenNftOracle.Message[] stolenNftProofs;
            bool isPublicPool;
            bytes32[][] publicPoolProofs;
        }

```

#### Recommendation:

It should look like:

```solidity
File: src/EthRouter.sol

48:     struct Buy {
            address payable pool;
            address nft;
            bool isPublicPool;
            uint256[] tokenIds;
            uint256[] tokenWeights;
            PrivatePool.MerkleMultiProof proof;
            uint256 baseTokenAmount;

        }

58:     struct Sell {
            address payable pool;
            address nft;
            bool isPublicPool;
            uint256[] tokenIds;
            uint256[] tokenWeights;
            PrivatePool.MerkleMultiProof proof;
            IStolenNftOracle.Message[] stolenNftProofs;
            bytes32[][] publicPoolProofs;
        }

```

### [GAS-20] USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS

#### Description:

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional `MLOAD` rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

262:             Change memory _change = changes[i];

```

```solidity
File: src/Factory.sol

82:         uint256[] memory tokenIds, // put in memory to avoid stack too deep error

82:         uint256[] memory tokenIds, // put in memory to avoid stack too deep error

161:     function tokenURI(uint256 id) public view override returns (string memory) {

```

```solidity
File: src/PrivatePool.sol

305:         IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error

305:         IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error

386:         uint256[] memory inputTokenIds,

387:         uint256[] memory inputTokenWeights,

388:         MerkleMultiProof memory inputProof,

389:         IStolenNftOracle.Message[] memory stolenNftProofs,

390:         uint256[] memory outputTokenIds,

391:         uint256[] memory outputTokenWeights,

392:         MerkleMultiProof memory outputProof

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

461:         (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

662:         uint256[] memory tokenIds,

663:         uint256[] memory tokenWeights,

664:         MerkleMultiProof memory proof

672:         bytes32[] memory leafs = new bytes32[](tokenIds.length);

```

```solidity
File: src/PrivatePoolMetadata.sol

17:     function tokenURI(uint256 tokenId) public view returns (string memory) {

19:         bytes memory metadata = abi.encodePacked(

35:     function attributes(uint256 tokenId) public view returns (string memory) {

39:         bytes memory _attributes = abi.encodePacked(

55:     function svg(uint256 tokenId) public view returns (bytes memory) {

59:         bytes memory _svg;

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

```

### [GAS-21] STRUCTS CAN BE PACKED INTO FEWER STORAGE SLOTS

#### Description:

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

#### **Proof Of Concept**

```solidity
File: src/EthRouter.sol

48:     struct Buy {
            address payable pool;
            address nft;
            uint256[] tokenIds;
            uint256[] tokenWeights;
            PrivatePool.MerkleMultiProof proof;
            uint256 baseTokenAmount;
            bool isPublicPool;
        }

58:     struct Sell {
            address payable pool;
            address nft;
            uint256[] tokenIds;
            uint256[] tokenWeights;
            PrivatePool.MerkleMultiProof proof;
            IStolenNftOracle.Message[] stolenNftProofs;
            bool isPublicPool;
            bytes32[][] publicPoolProofs;
        }

```

#### Recommendation:

It should look like:

```solidity
File: src/EthRouter.sol

48:     struct Buy {
            address payable pool;
            address nft;
            bool isPublicPool;
            uint256[] tokenIds;
            uint256[] tokenWeights;
            PrivatePool.MerkleMultiProof proof;
            uint256 baseTokenAmount;

        }

58:     struct Sell {
            address payable pool;
            address nft;
            bool isPublicPool;
            uint256[] tokenIds;
            uint256[] tokenWeights;
            PrivatePool.MerkleMultiProof proof;
            IStolenNftOracle.Message[] stolenNftProofs;
            bytes32[][] publicPoolProofs;
        }

```

### [GAS-22] TERNARY OPERATION IS CHEAPER THAN IF-ELSE STATEMENT

#### Description:

There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

110:    if (_baseToken == address(0)) {
            // transfer eth into the pool if base token is ETH
            address(privatePool).safeTransferETH(baseTokenAmount);
        } else {
            // deposit the base tokens from the caller into the pool
            ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);
        }


149:    if (token == address(0)) {
            msg.sender.safeTransferETH(amount);
        } else {
            ERC20(token).transfer(msg.sender, amount);
        }

```

```solidity
File: src/PrivatePool.sol

278:                if (baseToken != address(0)) {
                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);
                    } else {
                        recipient.safeTransferETH(royaltyFee);
                    }

345:                if (baseToken != address(0)) {
                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);
                    } else {
                        recipient.safeTransferETH(royaltyFee);
                    }

362:        if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
        } else {
            // transfer base tokens to the caller
            ERC20(baseToken).transfer(msg.sender, netOutputAmount);

426:       if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);
        } else {
            // check that the caller sent enough ETH to cover the fee amount and protocol fee
            if (msg.value < feeAmount + protocolFeeAmount) revert InvalidEthAmount();

522:     if (token == address(0)) {
            // transfer the ETH to the caller
            msg.sender.safeTransferETH(tokenAmount);
        } else {
            // transfer the tokens to the caller
            ERC20(token).transfer(msg.sender, tokenAmount);
        }
```

### [GAS-23] USAGE OF `UINT`/`INT` SMALLER THAN 32 BYTES (256 BITS) INCURS OVERHEAD

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

51:     uint16 public protocolFeeRate;

76:         uint56 _changeFee,

77:         uint16 _feeRate,

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

```

```solidity
File: src/PrivatePool.sol

58:     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

58:     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

66:     event SetFeeRate(uint16 feeRate);

88:     uint56 public changeFee;

91:     uint16 public feeRate;

162:         uint56 _changeFee,

163:         uint16 _feeRate,

562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {

606:         uint16 newFeeRate,

```

### [GAS-24] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: src/Factory.sol

161:     function tokenURI(uint256 id) public view override returns (string memory) {

```

```solidity
File: src/PrivatePoolMetadata.sol

17:     function tokenURI(uint256 tokenId) public view returns (string memory) {

30:         return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));

35:     function attributes(uint256 tokenId) public view returns (string memory) {

50:         return string(_attributes);

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

114:         return string(

```
