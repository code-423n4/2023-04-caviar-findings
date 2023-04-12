# Gas Optimization


## Summary
| No |  issue          |       instances |
|----|----------- | ------ |   
|[G-01]| abi.encodePacked() efficient then  abi.encode() | 2
|[G-02]|constructor Setting to payable | 2
|[G-03]| instead of && using nested if | 8
|[G-04]| public functions to external   | 2
|[G-05]| Functions guaranteed to revert when called by normal users can be marked payable | 6
|[G-06]| <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES | 4
|[G-07]| Make 3 event parameters indexed when possible | 2
|[G-08]| Use of Bit shift operators | 12
|[G-09]| Change public state variable visibility to private | 2
|[G-10]|Access mappings directly rather than using accessor functions | 2


## Deteils

### [G-01] abi.encodePacked() efficient then  abi.encode()

```solidity

177     abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L177

```solidity
675     leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L675

### [G-2] constructor Setting to payable

```solidity

90     constructor(address _royaltyRegistry) {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90

```solidity

53     constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#53


### [G-03] instead of && using nested if

```solidity
154     if (block.timestamp > deadline && deadline != 0) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154

```solidity
228     if (block.timestamp > deadline && deadline != 0) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228

```solidity
256     if (block.timestamp > deadline && deadline != 0) {  
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256


```solidity

87       if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {    

```    
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87

```solidity
277     if (royaltyFee > 0 && recipient != address(0)) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277

```solidity
344     if (royaltyFee > 0 && recipient != address(0)) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344

```solidity
397     if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397

```solidity
635     if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635

### [G-04] public functions to external
Saves having to do two JUMP instructions, along with stack setup
Istead of ownerOf() use idToOwner[]

```solidity

128     if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L128


```solidity

723     try ERC721(token).ownerOf(tokenId) returns (address result) {    
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#723

### [G-05] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
129     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129

```solidity
135     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135


```solidity
141     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141



```solidity
514     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514

```solidity
550     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550

```solidity
587     function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587



### [G-06] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES

```solidity
230     virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230


```solidity
231     virtualNftReserves -= uint128(weightSum);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231



```solidity
323     virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323



```solidity
324      virtualNftReserves += uint128(weightSum);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324

### [G-07] Make 3 event parameters indexed when possible

```solidity

58     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#58


```solidity
62     event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L62


### [G-08] Use of Bit shift operators

Check if arithmethic operations can be achieved using bitwise operators, if yes, implement same operations using bitwise operators and compare the gas consumed. Usually bitwise logic will be cheaper


```solidity
115      uint256 salePrice = inputAmount / buys[i].tokenIds.length;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#115

```solidity
182      uint256 salePrice = outputAmount / sells[i].tokenIds.length;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#182


```solidity
236     uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L236



```solidity
335     uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335



```solidity
668     return tokenIds.length * 1e18;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L668

```solidity
719     uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L719


```solidity
721     protocolFeeAmount = outputAmount * Factory(factory).protocolFeeRate() / 10_000;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L721

```solidity
722     feeAmount = outputAmount * feeRate / 10_000;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L722


```solidity
734     uint256 feePerNft = changeFee * 10 ** exponent;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L734


```solidity
736     feeAmount = inputAmount * feePerNft / 1e18;       
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L736


```solidity      
737     protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L737



```solidity
745     return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L745

### [G-09] Change public state variable visibility to private

```solidity
45     address public privatePoolImplementation;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L45

```solidity
48     address public privatePoolMetadata;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L48


## [G-10] Access mappings directly rather than using accessor functions

```solidity
128     if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L128

```solidity
723     try ERC721(token).ownerOf(tokenId) returns (address result) {    
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#723
