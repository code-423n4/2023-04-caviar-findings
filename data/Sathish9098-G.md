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

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

143: constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
144:        factory = payable(_factory);
145:        royaltyRegistry = _royaltyRegistry;
146:        stolenNftOracle = _stolenNftOracle;
147:    }

175: baseToken = _baseToken;
176: nft = _nft;

```
[PrivatePool.sol#L143-L147](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L143-L147),[PrivatePool.sol#L175-L176](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L175-L176)

```solidity
FILE: 2023-04-caviar/src/Factory.sol

function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
        privatePoolMetadata = _privatePoolMetadata;
    }

function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }

```
[Factory.sol#L129-L137](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L137)


### Recommendation Code

```soldiity
FILE: 2023-04-caviar/src/Factory.sol

function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
       assembly {
            sstore(privatePoolImplementation.slot, _privatePoolImplementation)
        }
```
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

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

225: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
277: if (royaltyFee > 0 && recipient != address(0)) {
344: if (royaltyFee > 0 && recipient != address(0)) {
397: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
635: if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

```
[PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L225),[PrivatePool.sol#L277](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L277),[PrivatePool.sol#L344](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L344),[PrivatePool.sol#L397](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L397),[PrivatePool.sol#L635](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L635),[]()

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

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

160: uint128 _virtualBaseTokenReserves,
161: uint128 _virtualNftReserves,
162: uint56 _changeFee,
163: uint16 _feeRate,
```
[PrivatePool.sol#L160-L163](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L160-L163)
##

## [G-15] Splitting require() statements that use && saves gas

See [this issue](https://github.com/code-423n4/2022-01-xdefi-findings/issues/128) which describes the fact that there is a larger deployment gas cost, but with enough runtime calls, the change ends up being cheaper by 3 gas

```solidity
```


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


```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

134: receive() external payable {}

```
[PrivatePool.sol#L134](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L134)

##

## [G-18] Use assembly to check for address(0)

Saves 6 gas per instance

```solidity
FILE : 2023-04-caviar/src/Factory.sol

110: if (_baseToken == address(0)) {

149: if (token == address(0)) {

```
[Factory.sol#L110](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L110),[Factory.sol#L149](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L149)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

254: if (baseToken != address(0)) {
277: if (royaltyFee > 0 && recipient != address(0)) {
278: if (baseToken != address(0)) {
344: if (royaltyFee > 0 && recipient != address(0)) {
345: if (baseToken != address(0)) {
357: if (baseToken == address(0)) {
397: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
421: if (baseToken != address(0)) {
489: if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
500: if (baseToken != address(0)) {
522: if (token == address(0)) {
635: if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
651:  if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

```
[PrivatePool.sol#L254](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L254)


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

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

238: for (uint256 i = 0; i < tokenIds.length; i++) {
272: for (uint256 i = 0; i < tokenIds.length; i++) {
329: for (uint256 i = 0; i < tokenIds.length; i++) {
441: for (uint256 i = 0; i < inputTokenIds.length; i++) {
496: for (uint256 i = 0; i < tokenIds.length; i++) {
518: for (uint256 i = 0; i < tokenIds.length; i++) {
673: for (uint256 i = 0; i < tokenIds.length; i++) {

```
[PrivatePool.sol#L238](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L238),[PrivatePool.sol#L272](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L272),[PrivatePool.sol#L329](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L329),[PrivatePool.sol#L441](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L441),[PrivatePool.sol#L446](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L446),[PrivatePool.sol#L496](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L496),[PrivatePool.sol#L518](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L518),[PrivatePool.sol#L673](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L673)



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

58: event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);
59: event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
60: event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
61: event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);
62: event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);
63: event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
64: event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);
65: event SetMerkleRoot(bytes32 merkleRoot);
66: event SetFeeRate(uint16 feeRate);
67: event SetUseStolenNftOracle(bool useStolenNftOracle);
68: event SetPayRoyalties(bool payRoyalties);

```
[PrivatePool.sol#L58-L68](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L58-L68)

##

## [G-23] Public functions not called by contract can declare as external to save gas 

External call cost is less expensive than of public functions.
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents’ functions and change the visibility from external to public.
The following functions could be set external to save gas and improve code quality:


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

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

240: ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
256: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);
259: if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
279: ERC20(baseToken).safeTransfer(recipient, royaltyFee);
331: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
346: ERC20(baseToken).safeTransfer(recipient, royaltyFee);
365: ERC20(baseToken).transfer(msg.sender, netOutputAmount);
423: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);
442: ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
447: ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
497: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
502: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);
519: ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
527: ERC20(token).transfer(msg.sender, tokenAmount);
638: ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);
648: ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);
651: if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);
765: try ERC721(token).ownerOf(tokenId) returns (address result) {
784: address lookupAddress = IRoyaltyRegistry(royaltyRegistry).getRoyaltyLookupAddress(nft);
786: if (IERC2981(lookupAddress).supportsInterface(type(IERC2981).interfaceId)) {
788: (recipient, royaltyFee) = IERC2981(lookupAddress).royaltyInfo(tokenId, salePrice);
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

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

254: if (baseToken != address(0)) {
278: if (baseToken != address(0)) {
345: if (baseToken != address(0)) {
421: if (baseToken != address(0)) {
500: if (baseToken != address(0)) {
651:  if (baseToken != address(0)) 

277: if (royaltyFee > 0 && recipient != address(0)) {
344: if (royaltyFee > 0 && recipient != address(0)) {

```
[PrivatePool.sol#L254](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L254)

##

## [G-26] <x> += <y> costs more gas than <x> = <x> + <y> for state variables (SAME FOR -=)

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

230: virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
231: virtualNftReserves -= uint128(weightSum);

323: virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
324: virtualNftReserves += uint128(weightSum);

```
[PrivatePool.sol#L230-L231](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L230-L231),[PrivatePool.sol#L323-L324](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L323-L324)

##

## [G-27] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement

```
require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
```
```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

268:  if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);

if (msg.value > feeAmount + protocolFeeAmount) {
                msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);
            }

```
[PrivatePool.sol#L268](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L268),[PrivatePool.sol#L435-L437](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L435-L437)

##

## [G-28] Don't declare variables inside the loops 

Declare the variables outside the loops and use inside 

(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L106-L115)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L261-L262)













public functions not called by contract
events should use 3 indexed rule 

GAS-1	Using bools for storage incurs overhead	3
GAS-2	Cache array length outside of loop	19
GAS-3	Don't initialize variables with default value	21
GAS-4	Pre-increments and pre-decrements are cheaper than post-increments and post-decrements	19
GAS-5	Use storage instead of memory for structs/arrays	5
GAS-6	Increments can be unchecked in for-loops	19
GAS-7	Use != 0 instead of > 0 for unsigned integer comparison	17