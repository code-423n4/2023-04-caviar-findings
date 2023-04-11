## GAS OPTIMIZATIONS

| Gas Count     | Issues | Instances | Gas Saved| 
| --------------- | --------------- | --------------- | --------------- | 
| [G-1] | Structs can be packed into fewer storage slots | 2 | 40000 |
| [G-2] | Optimize names to save gas | 38 | - |
| [G-3] | Functions guaranteed to revert when called by normal users can be marked payable | 10 | 210 |
| [G-4] | Setting the constructor to payable | 2 | 26 |
| [G-5] | Use assembly to write address storage values | 8 | - |
| [G-6] | Use nested if and, avoid multiple check combinations | 9 | 81 |
| [G-7] | Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 7 | 196 |
| [G-8] | Empty blocks should be removed or emit something | 4 | - |
| [G-9] | Use assembly to check for address(0) | 15 | 90 |
| [G-10] | ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops | 18 | 2520 |
| [G-11] | Amounts should be checked for 0 before calling a transfer functions | 16 | - |
| [G-12] | Use 3 indexed parameters rule for events to save gas | 11 | - |
| [G-13] | public functions to external | 23 | 345 |
| [G-14] | Avoid contract existence checks by using low level calls | 30 | 3000 |
| [G-15] | Duplicated require()/if() checks should be refactored to a modifier or function | 12 | - |
| [G-16] | <x> += <y> costs more gas than <x> = <x> + <y> for state variables (SAME FOR -=) | 4 | 452 |
| [G-17] | Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement | 2 | 440 |
| [G-18] | Don't declare variables inside the loops  | 3 | - |
| [G-19] | The condition check should be top of the functions  | 1 | - |
| [G-20] | Cheaper input validations should come before expensive operations | 1 | - |
| [G-21] | Sort Solidity operations using short-circuit mode | 8 | - |
| [G-22] | Multiplication/division by 2 should use bit shifting | 3 | - |
| [G-23] | abi.encode() is less efficient than abi.encodePacked() | 2 | - |
| [G-24] | State variables with values known at compile time should be constants | 1 | - |
| [G-25] | State variables should be cached in stack variables rather than re-reading them from storage | 25 | 2500 |
| [G-26] | With assembly, .call (bool success) transfer can be done gas-optimized | 1 | -|

##

## [G-1] Structs can be packed into fewer storage slots

> Instances(2)

> Approximate Gas Saved : 40000 (2 slots) gas

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

## [G-2] Optimize names to save gas

> Instances(38)

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
## [G-3] Functions guaranteed to revert when called by normal users can be marked payable

> Instances(10)

> Approximate Gas Saved : 210 gas

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

```solidity
FILE: 2023-04-caviar/src/Factory.sol

129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
148: function withdraw(address token, uint256 amount) public onlyOwner {
```
[Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

514:  function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
538:  function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
550:  function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
562:  function setFeeRate(uint16 newFeeRate) public onlyOwner {
576:  function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
587:  function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```
[PrivatePool.sol#L538](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538),[PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L550),[PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L562),[PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L576),[PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L587)

##

## [G-4] Setting the constructor to payable

> Instances(2)

> Approximate Gas Saved : 26 gas

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

## [G-5] Use assembly to write address storage values

> Instances(8)

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

```solidity
FILE: 2023-04-caviar/src/Factory.sol

function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
       assembly {
            sstore(privatePoolImplementation.slot, _privatePoolImplementation)
        }
```
##

## [G-6] Use nested if and, avoid multiple check combinations

> Instances(9)

> Approximate Gas Saved : 81 gas

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

[As per gas test](https://gist.github.com/sathishpic22/fe96671bafb22ceaace7fc05a66bd115) its possible to save 9 gas 

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

## [G-7] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

> Instances(13)

> Approximate Gas Saved : 364 gas

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

(https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html)

Each operation involving a uint128 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint128, as well as the associated stack operations of doing so. Use a larger size then downcast where needed


```solidity
FILE: 2023-04-caviar/src/Factory.sol

74: uint128 _virtualBaseTokenReserves,
75: uint128 _virtualNftReserves,
76: uint56 _changeFee,
77: uint16 _feeRate,
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {


```
[Factory.sol#L74-L77](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L74-L77)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

112:  uint128 public virtualNftReserves;
104:  uint128 public virtualBaseTokenReserves;
91:   uint16 public feeRate;
88:   uint56 public changeFee;

160: uint128 _virtualBaseTokenReserves,
161: uint128 _virtualNftReserves,
162: uint56 _changeFee,
163: uint16 _feeRate,
```
[PrivatePool.sol#L160-L163](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L160-L163)

##

## [G-8] Empty blocks should be removed or emit something

> Instances(4)

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

## [G-9] Use assembly to check for address(0)

> Instances(15)

> Approximate Gas Saved : 90 gas

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

## [G-10] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

> Instances(17)

> Approximate Gas Saved : 2040 gas

The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher. 

[As per test](https://gist.github.com/sathishpic22/61f9b96d64c40b9c34824aa0f337e9d2) its possible to save 100-150 gas 

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

## [G-11] Amounts should be checked for 0 before calling a transfer functions 

> Instances(16)

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

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

256: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);
259: if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
279: ERC20(baseToken).safeTransfer(recipient, royaltyFee);
346: ERC20(baseToken).safeTransfer(recipient, royaltyFee);
365: ERC20(baseToken).transfer(msg.sender, netOutputAmount);
423: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);
502: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);
527: ERC20(token).transfer(msg.sender, tokenAmount);
638: ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);
648: ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);
651: if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);
```
[PrivatePool.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L256)

##

## [G-12] Use 3 indexed parameters rule for events to save gas

> Instances(12)

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

## [G-13] public functions to external

> Instances(23)

> Approximate Gas Saved : 345 gas

Its possible to save 10-15 gas using external instead public for every function call 

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

```solidity
FILE: 2023-04-caviar/src/Factory.sol

function create(
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
    ) public payable returns (PrivatePool privatePool) {

135:  function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148: function withdraw(address token, uint256 amount) public onlyOwner {

129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

168: function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

```

[Factory.sol#L71-L84](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L71-L84),[Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135),[Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141),[Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L148),[Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129),[Factory.sol#L168](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L168)

```solidity
FILE: 2023-04-caviar/src/PrivatePoolMetadata.sol

17: function tokenURI(uint256 tokenId) public view returns (string memory) {


```
[PrivatePoolMetadata.sol#L17](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L17)

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

99:  function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
152: function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public {

```
[EthRouter.sol#L99](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L99),[EthRouter.sol#L152](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L152),[]()

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

 function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {

function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {

459:  function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

484:  function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {

514:  function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538:  function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
550:  function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
562:  function setFeeRate(uint16 newFeeRate) public onlyOwner {
576:  function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
587:  function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```
[PrivatePool.sol#L211-L214](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211-L214),[PrivatePool.sol#L301-L306](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L301-L306),[PrivatePool.sol#L385-L393](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L385-L393),[PrivatePool.sol#L459](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L459),[PrivatePool.sol#L484](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L484),[PrivatePool.sol#L514](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L514),[PrivatePool.sol#L538](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538),[PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L550),[PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L562),[PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L576),[PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L587)

##

## [G-14] Avoid contract existence checks by using low level calls

> Instances(30)

> Approximate Gas Saved : 3000 gas

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

## [G-15] Duplicated require()/if() checks should be refactored to a modifier or function

> Instances(12)

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

## [G-16] <x> += <y> costs more gas than <x> = <x> + <y> for state variables (SAME FOR -=)

> Instances(4)

> Approximate Gas Saved : 452 gas

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

## [G-17] Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement

> Instances(2)

> Approximate Gas Saved : 442 gas

[As per test](https://gist.github.com/sathishpic22/255e4a8c2b9bbfd52bb5637a132bdf17) possible to save 221 gas when we use unchecked for subtractions 

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

## [G-18] Don't declare variables inside the loops 

> Instances(3)

Declare the variables outside the loops and use inside . Its possible to save 9 gas for every iterations of the for loops. The gas differs as per data types.

(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L106-L115)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L261-L262)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L329-L335)

##

## [G-19] The condition check should be top of the functions 

> Instances(1)

By performing these checks at the beginning of the function, the contract can avoid unnecessary processing and save gas

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

>  if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount(); Condition should be placed top of the function. Waste of gas if (baseToken != address(0) && msg.value > 0) condition failed after getting sumWeightsAndValidateProof(),buyQuote() variable values 

function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
    {
        // ~~~ Checks ~~~ //

        // calculate the sum of weights of the NFTs to buy
        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

        // calculate the required net input amount and fee amount
        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

        // check that the caller sent 0 ETH if the base token is not ETH
        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

```
[PrivatePool.sol#L211-L225](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211-L225)

```

##

## [G-20] Cheaper input validations should come before expensive operations

> Instances(1)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

> "if (useStolenNftOracle) {" condition consumes less gas than "if (baseToken != address(0) && msg.value > 0) " condition check 

397: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

400: if (useStolenNftOracle) {
            IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, inputTokenIds, stolenNftProofs);
        }

```
[PrivatePool.sol#L397-L402](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L397-L402)

##

## [G-21] Sort Solidity operations using short-circuit mode

> Instances(7)

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 
//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)

```
```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

it is generally more gas efficient to use a local variable to store the deadline time in a condition check, rather than calling block.timestamp directly in the condition. This is because accessing the block.timestamp requires an external call to the Ethereum network, which is more expensive in terms of gas compared to using a local variable

101: if (block.timestamp > deadline && deadline != 0) {
154: if (block.timestamp > deadline && deadline != 0) {
228: if (block.timestamp > deadline && deadline != 0) {
256: if (block.timestamp > deadline && deadline != 0) {

```
[EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L101)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

msg.value > 0 check this condition first then baseToken != address(0). Because baseToken is state variable 

225: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
397: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
489: if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
635: if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
```
[PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L225)

##

## [G-22] Multiplication/division by 2 should use bit shifting

> Instances(3)

> Approximate Gas Saved : 15 gas

<x> * 2 is equivalent to <x> << 1 and <x> / 2 is the same as <x> >> 1. The MUL and DIV opcodes cost 5 gas, whereas SHL and SHR only cost 3 gas

```solidity 
FILE: FILE: 2023-04-caviar/src/PrivatePool.sol

668:  return tokenIds.length * 1e18;
736: feeAmount = inputAmount * feePerNft / 1e18;
734:  uint256 feePerNft = changeFee * 10 ** exponent;

```
[PrivatePool.sol#L668](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L668)

##

## [G-23] abi.encode() is less efficient than abi.encodePacked()

> Instances(2)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

675: leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

```
[PrivatePool.sol#L675](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L675)

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

177: abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))

```
[EthRouter.sol#L177](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L177)

##

## [G-24] State variables with values known at compile time should be constants

> Instances(1)

Variables with values known at compile time and that do not change at runtime should be declared as constant.

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

changeFee can be declared as constant since no changes in contract. The value assigned only one time through out the contract 

179: changeFee = _changeFee;
```
[PrivatePool.sol#L179](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L179)

##

## [G-25] State variables should be cached in stack variables rather than re-reading them from storage

> Instances(25)

> Approximate Gas Saved : 2500 gas

Caching will replace each Gwarmaccess (100 gas) with a much cheaper stack read.
Less obvious fixes/optimizations include having local storage variables of mappings within state variable mappings or mappings within state variable structs, having local storage variables of structs within mappings, having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

```solidity
2023-04-caviar/src/PrivatePool.sol

baseToken should be cached with stack variable 

buy() function

225:  if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
256:  ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);
259:  if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
278:  if (baseToken != address(0)) {

sell() function

345: if (baseToken != address(0)) {
365: ERC20(baseToken).transfer(msg.sender, netOutputAmount);
368: if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

change() function

397:  if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
421:  if (baseToken != address(0)) {
423:  ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);
429:   if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);

deposit() function

489: if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
500: if (baseToken != address(0)) {
502: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);

flashLoan() function

635: if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
651: if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

733: uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;
744: uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());

payRoyalties should be cached with stack variable 

buy() function

242: if (payRoyalties) {
271: if (payRoyalties) {

nft should be cached with stack variable

buy() function

317: IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
331: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

change() function

401: IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, inputTokenIds, stolenNftProofs);
442: ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
447: ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);

```
[PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L225)

##

## [G-26] With assembly, .call (bool success) transfer can be done gas-optimized

> Instances(1)

Return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

(https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

461: (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

```
[PrivatePool.sol#L461](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L461)










GAS-1	Using bools for storage incurs overhead	3
GAS-2	Cache array length outside of loop	19
GAS-3	Don't initialize variables with default value	21
GAS-4	Pre-increments and pre-decrements are cheaper than post-increments and post-decrements	19
GAS-5	Use storage instead of memory for structs/arrays	5
GAS-6	Increments can be unchecked in for-loops	19
GAS-7	Use != 0 instead of > 0 for unsigned integer comparison	17