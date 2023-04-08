## GAS OPTIMIZATIONS

##

## [G-1] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

```solidity
FILE : 2023-04-rubicon/contracts/V2Migrator.sol

/// @audit v1ToV2Pools (constructor)
33: v1ToV2Pools[bathTokensV1[i]] = bathTokensV2[i];
```
[V2Migrator.sol#L33](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/V2Migrator.sol#L33)

```solidity
FILE : 2023-04-rubicon/contracts/utilities/poolsUtility/Position.sol

/// @audit oracle (constructor),rubiconMarket (constructor),bathHouseV2 (constructor),comptroller (constructor)
55: oracle = PriceOracle(_oracle);
56: rubiconMarket = RubiconMarket(_rubiconMarket);
57: bathHouseV2 = BathHouseV2(_bathHouseV2);
58: comptroller = bathHouseV2.comptroller();

```
[Position.sol#L55-L58](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/utilities/poolsUtility/Position.sol#L55-L58)


##

## [G-2] State variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

```solidity

FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

219: uint256 public last_offer_id; // 32 bytes  SLOT-1
222: mapping(uint256 => OfferInfo) public offers; // 32 bytes  SLOT-2
224: bool locked; // 1 bytes  SLOT-3
227: uint256 internal feeBPS; // 32 bytes  SLOT-4
230: address internal feeTo; // 20 bytes   SLOT-5

> Currently total of 5 slots occupied 

```
[RubiconMarket.sol#L219-L230](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L219-L230)

### After Mitigations 

```solidity

FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

  219: uint256 public last_offer_id; // 32 bytes  SLOT-1
  222: mapping(uint256 => OfferInfo) public offers; // 32 bytes  SLOT-2
- 224: bool locked; // 1 bytes  
  227: uint256 internal feeBPS; // 32 bytes  SLOT-3
  230: address internal feeTo; // 20 bytes   SLOT-4
+ 224: bool locked; // 1 bytes  SLOT-4

```
### After mitigation total slots 4 . Saved 1 slot and saves Gsset (20000 gas)

##

## [G-3] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

/// @audit Variable ordering with 6 slots instead of the current 7 :
///  address(20):pool,address(20):nft,bool(1):isPublicPool,uint256(dynamic):tokenIds,uint256(dynamic):tokenWeights, PrivatePool(32):proof,uint256(32):baseTokenAmount
 48:  struct Buy {
 49:       address payable pool;
 50:       address nft;
+55:       bool isPublicPool;
 51:       uint256[] tokenIds;
 52:       uint256[] tokenWeights;
 53:       PrivatePool.MerkleMultiProof proof;
 54:       uint256 baseTokenAmount;
-55:       bool isPublicPool;
 56:    }
```
[EthRouter.sol#L48-L56](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L48-L56)

```solidity
/// @audit Variable ordering with 7 slots instead of the current 8 :
///   
 58: struct Sell {
 59:       address payable pool;
 60:       address nft;
+65:       bool isPublicPool;
 61:       uint256[] tokenIds;
 62:       uint256[] tokenWeights;
 63:       PrivatePool.MerkleMultiProof proof;
 64:       IStolenNftOracle.Message[] stolenNftProofs;
-65:       bool isPublicPool;
 66:       bytes32[][] publicPoolProofs;
 67:   }

```
[EthRouter.sol#L58-L67](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L58-L67)

## [G-4] Use a more recent version of solidity

One of the main ways that Solidity reduces gas costs is through the use of more efficient bytecode. The Solidity compiler generates optimized bytecode that uses fewer instructions and consumes less gas than previous versions. This optimization can result in significant gas savings, particularly for contracts with complex logic or large data structures

- Inline assembly
- Storage layout optimization
- Better gas estimation
-Low-level data access

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

3: pragma solidity ^0.8.9;
```

```solidity
FILE : 2023-04-rubicon/contracts/periphery/BathBuddy.sol

2: pragma solidity ^0.8.0;
```

``` solidity
FILE : 2023-04-rubicon/contracts/utilities/poolsUtility/Position.sol

2: pragma solidity 0.8.17;
```
##

## [G-5] Optimize names to save gas

public/external function names and public member variable names can be optimized to save gas. See this [link](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, [per sorted position shifted](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

> public/external functions

```solidity
FILE : 2023-04-caviar/src/Factory.sol

/// @audit create(),setPrivatePoolMetadata(),setPrivatePoolImplementation(),setProtocolFeeRate(),withdraw(),predictPoolDeploymentAddress()
37: contract Factory is ERC721, Owned {

FILE : 2023-04-caviar/src/PrivatePoolMetadata.sol

/// @audit  tokenURI(),attributes(),svg()
14: contract PrivatePoolMetadata {

FILE : 2023-04-caviar/src/EthRouter.sol

/// @audit buy(),sell(),deposit(),change(),getRoyalty()
45:contract EthRouter is ERC721TokenReceiver {

FILE: 2023-04-caviar/src/PrivatePool.sol

/// @audit initialize(),buy(),sell(),change(),execute(),deposit(),withdraw(),setVirtualReserves(),setMerkleRoot(),setFeeRate(),setUseStolenNftOracle(),setPayRoyalties(),setAllParameters(),flashLoan(),sumWeightsAndValidateProof(),buyQuote(),sellQuote(),changeFeeQuote(),price(),flashFee(),flashFeeToken(),availableForFlashLoan(),
45: contract PrivatePool is ERC721TokenReceiver {

```
##

## [G-7] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```solidity
FILE: 2023-04-caviar/src/Factory.sol

129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
148: function withdraw(address token, uint256 amount) public onlyOwner {
```
[Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129)

##

## [G-8] Do not calculate constants

Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas

```solidity
```

##

## [G-9] internal/Private functions or Modifiers only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

```solidity 
```

##

## [G-10] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

It is true that not using the named return variables when a function returns wastes deployment gas.

```solidity

```

##

## [G-11] Setting the constructor to payable

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks

```solidity
FILE: 2023-04-caviar/src/Factory.sol

53: constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
```
[Factory.sol#L53](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L53)

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

90: constructor(address _royaltyRegistry) {
```


##

## [G-12] Use assembly to write address storage values

```solidity
FILE : 2023-04-caviar/src/EthRouter.sol

91: royaltyRegistry = _royaltyRegistry;
```
[EthRouter.sol#L91](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L91)

##

## [G-13] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

101: if (block.timestamp > deadline && deadline != 0) {
154: if (block.timestamp > deadline && deadline != 0) {
228: if (block.timestamp > deadline && deadline != 0) {
256: if (block.timestamp > deadline && deadline != 0) {

```
[EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L101),[EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L154),[EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L228),[EthRouter.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L256)

##

## [G-14] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

(https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)

```solidity
FILE: 2023-04-caviar/src/Factory.sol

74: uint128 _virtualBaseTokenReserves,
75: uint128 _virtualNftReserves,
76: uint56 _changeFee,
77: uint16 _feeRate,

```
[Factory.sol#L74-L77](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L74-L77)
##

## [G-15] Splitting require() statements that use && saves gas

See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by 3 gas

```solidity
```

##

## [G-16] Add unchecked {} for subtractions where the operands cannot underflow

```solidity
FILE : 2023-04-rubicon/contracts/RubiconMarket.sol

The block.number is not going to less than 10

943: _rank[id].delb < block.number - 10
```
[RubiconMarket.sol#L943](https://github.com/code-423n4/2023-04-rubicon/blob/511636d889742296a54392875a35e4c0c4727bb7/contracts/RubiconMarket.sol#L943)

##

## [G-17] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}). Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas

```solidity
FILE : 2023-04-caviar/src/Factory.sol

53: constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
55: receive() external payable {}
```
[Factory.sol#L53-L55](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L53-L55)

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

88: receive() external payable {}

```
[EthRouter.sol#L88](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L88)

##

## [G-18] Use assembly to check for address(0)

Saves 6 gas per instance

```solidity
FILE : 2023-04-caviar/src/Factory.sol

110: if (_baseToken == address(0)) {

149: if (token == address(0)) {

```
[Factory.sol#L110](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L110),[Factory.sol#L149](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L149)

##

## [G-19] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are

```solidity
FILE : 2023-04-caviar/src/Factory.sol

119: for (uint256 i = 0; i < tokenIds.length; i++) {

```
[Factory.sol#L119](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L119)

```solidity
FILE : 2023-04-caviar/src/EthRouter.sol

106: for (uint256 i = 0; i < buys.length; i++) {
116: for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
134: for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
159: for (uint256 i = 0; i < sells.length; i++) {
161: for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
239: for (uint256 i = 0; i < tokenIds.length; i++) {
261: for (uint256 i = 0; i < changes.length; i++) {
265: for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
284: for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
```
[EthRouter.sol#L106](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L106),[EthRouter.sol#L116](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L116),[EthRouter.sol#L134](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L134),[thRouter.sol#L159-L161](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L159-L161),[EthRouter.sol#L239](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L239),[EthRouter.sol#L261](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L261),[EthRouter.sol#L284](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L284)
##

## [G-20] Amounts should be checked for 0 before calling a transfer functions 

Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it’s not consistently done in the solution.
I suggest adding a non-zero-value check here

```solidity
FILE : 2023-04-caviar/src/Factory.sol

152: ERC20(token).transfer(msg.sender, amount);
150: msg.sender.safeTransferETH(amount);
120: ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
115: ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);
112: address(privatePool).safeTransferETH(baseTokenAmount);

```
[Factory.sol#L150-L152](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L150-L152),[Factory.sol#L112](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L112)

##

## [G-21] Use 3 indexed parameters rule for events to save gas

Its must add 3 indexed parameter for every event. If event has less than 3 parameters all parameters should be indexed 

```solidity
FILE : 2023-04-caviar/src/Factory.sol

41: event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);
```

[Factory.sol#L41](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L41)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol


()
()

##

## [G-23] Public functions not called by contract can declare as external 

```solidity
FILE : 2023-04-caviar/src/PrivatePoolMetadata.sol

17: function tokenURI(uint256 tokenId) public view returns (string memory) {
35: function attributes(uint256 tokenId) public view returns (string memory) {
55: function svg(uint256 tokenId) public view returns (bytes memory) {

```
[PrivatePoolMetadata.sol#L35](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L35)

##

## [G-24] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

136: ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
166: ERC721(sells[i].nft).setApprovalForAll(sells[i].pool, true);
162: ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
240: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
244: ERC721(nft).setApprovalForAll(privatePool, true);
266: ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);
270: ERC721(_change.nft).setApprovalForAll(_change.pool, true);
285: ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);
311: (recipient, royaltyFee) = IERC2981(lookupAddress).royaltyInfo(tokenId, salePrice);
```

##

## [G-25] Duplicated require()/if() checks should be refactored to a modifier or function

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

101: if (block.timestamp > deadline && deadline != 0) {
154: if (block.timestamp > deadline && deadline != 0) {
228: if (block.timestamp > deadline && deadline != 0) {
256: if (block.timestamp > deadline && deadline != 0) {

```
[EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L101),[EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L154),[EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L228),[EthRouter.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L256)













public functions not called by contract
events should use 3 indexed rule 

GAS-1	Using bools for storage incurs overhead	3
GAS-2	Cache array length outside of loop	19
GAS-3	Don't initialize variables with default value	21
GAS-4	Pre-increments and pre-decrements are cheaper than post-increments and post-decrements	19
GAS-5	Use storage instead of memory for structs/arrays	5
GAS-6	Increments can be unchecked in for-loops	19
GAS-7	Use != 0 instead of > 0 for unsigned integer comparison	17