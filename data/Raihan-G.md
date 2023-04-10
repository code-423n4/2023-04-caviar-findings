# Gas Optimization 

# Summary

|   | Issue | Instance 
| - | ----- | -------
|[G-01]|  Use double if statements instead of && | 11
|[G-02]|  Make 3 event parameters indexed when possible | 3
|[G-03]| public functions to external | 1
|[G-04]| Access mappings directly rather than using accessor functions | 2
|[G-05]| Amounts should be checked for 0 before calling a transfer | 2
|[G-06]| abi.encode() is less efficient than abi.encodePacked() | 2
|[G-07]| Functions guaranteed to revert when called by normal users can be marked payable | 9
|[G-08]| Change public state variable visibility to private | 2
|[G-09]| <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES | 4
|[G-10]| Use of Bit shift operators | 12
|[G-11]| Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead | 4
|[G-12]| Setting the constructor to payable | 3

## [G-01] Use double if statements instead of &&
 If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.

```solidity
File: /tree/main/src/EthRouter.sol

101     if (block.timestamp > deadline && deadline != 0) {
154     if (block.timestamp > deadline && deadline != 0) {
228     if (block.timestamp > deadline && deadline != 0) {
256     if (block.timestamp > deadline && deadline != 0) {    
``` 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
File: /tree/main/src/Factory.sol

87       if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {    

```    
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87

```solidity
File: /tree/main/src/PrivatePool.sol

225     if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

277     if (royaltyFee > 0 && recipient != address(0)) {

344     if (royaltyFee > 0 && recipient != address(0)) {

397     if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

489     if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

635     if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-02] Make 3 event parameters indexed when possible
 It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed

```solidity
File: Factory.sol

41     event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#41

```solidity
File: /tree/main/src/PrivatePool.sol

58     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

62     event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-03] public functions to external
 External call cost is less expensive than of public functions.
Contracts are allowed to override their parents’ functions and change the visibility from external to public.

```solidity
File: /tree/main/src/Factory.sol

161     function tokenURI(uint256 id) public view override returns (string memory) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L161

## [G-04] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup
Istead of ownerOf() use idToOwner[]

```solidity
File: /tree/main/src/PrivatePool.sol

128     if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
723     try ERC721(token).ownerOf(tokenId) returns (address result) {    
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-05] Amounts should be checked for 0 before calling a transfer
Checking non-zero transfer values can avoid an expensive external call and save gas.
While this is done at some places, it's not consistently done in the solution.

```solidity
File: /tree/main/src/Factory.sol

152      ERC20(token).transfer(msg.sender, amount);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L152

```solidity
File: /tree/main/src/PrivatePool.sol

527      ERC20(token).transfer(msg.sender, tokenAmount);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L527

## [G-06] abi.encode() is less efficient than abi.encodePacked()

```solidity
File: /tree/main/src/EthRouter.sol

177     abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L177

```solidity
File: /tree/main/src/PrivatePool.sol

675     leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L675

## [G-07] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
File: /tree/main/src/Factory.sol

129     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148     function withdraw(address token, uint256 amount) public onlyOwner {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
File: /tree/main/src/PrivatePool.sol

514     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

550     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562     function setFeeRate(uint16 newFeeRate) public onlyOwner {

576     function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587     function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-08] Change public state variable visibility to private

```solidity
File: /tree/main/src/EthRouter.sol

45     address public privatePoolImplementation;

48     address public privatePoolMetadata;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

## [G-9] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

```solidity
File: /tree/main/src/PrivatePool.sol

230     virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231     virtualNftReserves -= uint128(weightSum);

323     virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324      virtualNftReserves += uint128(weightSum);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230

## [G-10] Use of Bit shift operators
Check if arithmethic operations can be achieved using bitwise operators, if yes, implement same operations using bitwise operators and compare the gas consumed. Usually bitwise logic will be cheaper
Exmple:

- uint a = b / 2
+ uint a = b >> 2
  
- uint a = b * 2
+ uint a = b << 2  
```solidity
File: /tree/main/src/PrivatePool.sol

236     uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;

335     uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;

668     return tokenIds.length * 1e18;

719     uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);

721     protocolFeeAmount = outputAmount * Factory(factory).protocolFeeRate() / 10_000;

722     feeAmount = outputAmount * feeRate / 10_000;

734     uint256 feePerNft = changeFee * 10 ** exponent;

736     feeAmount = inputAmount * feePerNft / 1e18;       

737     protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;

745     return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

```solidity
File: /tree/main/src/EthRouter.sol

115      uint256 salePrice = inputAmount / buys[i].tokenIds.length;

182      uint256 salePrice = outputAmount / sells[i].tokenIds.length;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

## [G-11] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
 When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

 ```solidity
 File: /tree/main/src/EthRouter.sol
 
 48       struct Buy {
        address payable pool;
        address nft;
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        uint256 baseTokenAmount;
        bool isPublicPool;
    }

58  struct Sell {
        address payable pool;
        address nft;
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        IStolenNftOracle.Message[] stolenNftProofs;
        bool isPublicPool;
        bytes32[][] publicPoolProofs;
    }

69  struct Change {
        address payable pool;
        address nft;
        uint256[] inputTokenIds;
        uint256[] inputTokenWeights;
        PrivatePool.MerkleMultiProof inputProof;
        IStolenNftOracle.Message[] stolenNftProofs;
        uint256[] outputTokenIds;
        uint256[] outputTokenWeights;
        PrivatePool.MerkleMultiProof outputProof;
    }
 ```
 https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

 ```solidity
 File: /tree/main/src/interfaces/IStolenNftOracle.sol

6   struct Message {
        bytes32 id;
        bytes payload;
        // The UNIX timestamp when the message was signed by the oracle
        uint256 timestamp;
        // ECDSA signature or EIP-2098 compact signature
        bytes signature;
    }
 ```
 https://github.com/code-423n4/2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol#L6

## [G-12] Setting the constructor to payable
You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

```solidity
File: /tree/main/src/EthRouter.sol

90     constructor(address _royaltyRegistry) {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90

```solidity
File: /tree/main/src/Factory.sol

53     constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#53

```solidity
File: /tree/main/src/PrivatePool.sol
143     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#143