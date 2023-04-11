
### [G01] State variables only set in the constructor should be declared `immutable`

#### Impact
Avoids a Gusset (20000 gas)

#### Findings:
```
2023-04-caviar/src/EthRouter.sol::220 => address payable privatePool,
2023-04-caviar/src/Factory.sol::45 => address public privatePoolImplementation;
2023-04-caviar/src/Factory.sol::48 => address public privatePoolMetadata;
2023-04-caviar/src/Factory.sol::51 => uint16 public protocolFeeRate;
2023-04-caviar/src/PrivatePool.sol::82 => address public baseToken;
2023-04-caviar/src/PrivatePool.sol::85 => address public nft;
2023-04-caviar/src/PrivatePool.sol::88 => uint56 public changeFee;
2023-04-caviar/src/PrivatePool.sol::91 => uint16 public feeRate;
2023-04-caviar/src/PrivatePool.sol::94 => bool public initialized;
2023-04-caviar/src/PrivatePool.sol::97 => bool public payRoyalties;
2023-04-caviar/src/PrivatePool.sol::100 => bool public useStolenNftOracle;
2023-04-caviar/src/PrivatePool.sol::104 => uint128 public virtualBaseTokenReserves;
2023-04-caviar/src/PrivatePool.sol::112 => uint128 public virtualNftReserves;
2023-04-caviar/src/PrivatePool.sol::116 => bytes32 public merkleRoot;
```




### [G02] Not using the named return variables when a function returns, wastes deployment gas


#### Findings:
```
2023-04-caviar/src/PrivatePoolMetadata.sol::109 => return _svg;
```


### [G03] It costs more gas to initialize variables to zero than to let the default of zero be applied

#### Impact
Issue Information: [G011](https://github.com/Bnke0x0/c4-common-issues/blob/main/0-Gas-Optimizations.md#g011---It-costs-more-gas-to-initialize-variables-to-zero-than-to-let-the-default-of-zero-be-applied)

#### Findings:
```
2023-04-caviar/src/EthRouter.sol::106 => for (uint256 i = 0; i < buys.length; i++) {
2023-04-caviar/src/EthRouter.sol::116 => for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::134 => for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::159 => for (uint256 i = 0; i < sells.length; i++) {
2023-04-caviar/src/EthRouter.sol::161 => for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::183 => for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::239 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/EthRouter.sol::261 => for (uint256 i = 0; i < changes.length; i++) {
2023-04-caviar/src/EthRouter.sol::265 => for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::284 => for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
2023-04-caviar/src/Factory.sol::119 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::237 => uint256 royaltyFeeAmount = 0;
2023-04-caviar/src/PrivatePool.sol::238 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::272 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::328 => uint256 royaltyFeeAmount = 0;
2023-04-caviar/src/PrivatePool.sol::329 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::441 => for (uint256 i = 0; i < inputTokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::446 => for (uint256 i = 0; i < outputTokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::496 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::518 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::673 => for (uint256 i = 0; i < tokenIds.length; i++) {
```



### [G04] `++i` costs less gas than `i++`, especially when it’s used in forloops (`--i`/`i--` too)


#### Findings:
```
2023-04-caviar/src/EthRouter.sol::106 => for (uint256 i = 0; i < buys.length; i++) {
2023-04-caviar/src/EthRouter.sol::116 => for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::134 => for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::159 => for (uint256 i = 0; i < sells.length; i++) {
2023-04-caviar/src/EthRouter.sol::161 => for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::183 => for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::239 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/EthRouter.sol::261 => for (uint256 i = 0; i < changes.length; i++) {
2023-04-caviar/src/EthRouter.sol::265 => for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
2023-04-caviar/src/EthRouter.sol::284 => for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
2023-04-caviar/src/Factory.sol::119 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::238 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::272 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::329 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::441 => for (uint256 i = 0; i < inputTokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::446 => for (uint256 i = 0; i < outputTokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::496 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::518 => for (uint256 i = 0; i < tokenIds.length; i++) {
2023-04-caviar/src/PrivatePool.sol::673 => for (uint256 i = 0; i < tokenIds.length; i++) {
```




### [G05] Duplicated `require()`/`revert()` checks should be refactored to a modifier or function


#### Findings:
```
2023-04-caviar/src/EthRouter.sol::102 => revert DeadlinePassed();
```


### [G06] Functions guaranteed to revert when called by normal users can be marked `payable`

#### Impact
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
#### Findings:
```
2023-04-caviar/src/Factory.sol::129 => function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
2023-04-caviar/src/Factory.sol::135 => function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
2023-04-caviar/src/Factory.sol::141 => function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
2023-04-caviar/src/Factory.sol::148 => function withdraw(address token, uint256 amount) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::514 => function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::538 => function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::550 => function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::562 => function setFeeRate(uint16 newFeeRate) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::576 => function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::587 => function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```





### [G07] Empty blocks should be removed or emit something

#### Impact
The code should be refactored such that they no longer exist, or the 
block should do something useful, such as emitting an event or 
reverting. If the contract is meant to be extended, the contract should 
be abstract and the function signatures be added without 
any default implementation. If the block is an empty if-statement block 
to avoid doing subsequent checks in the else-if/else conditions, the 
else-if/else conditions should be nested under the negation of the 
if-statement, because they involve different classes of checks, which 
may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}})
#### Findings:
```
2023-04-caviar/src/EthRouter.sol::88 => receive() external payable {}
2023-04-caviar/src/Factory.sol::53 => constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
2023-04-caviar/src/Factory.sol::55 => receive() external payable {}
2023-04-caviar/src/PrivatePool.sol::134 => receive() external payable {}
```



### [G08] `abi.encode()` is less efficient than abi.encodePacked()


#### Findings:
```
2023-04-caviar/src/EthRouter.sol::177 => abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))
2023-04-caviar/src/PrivatePool.sol::675 => leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
2023-04-caviar/src/PrivatePoolMetadata.sol::19 => bytes memory metadata = abi.encodePacked(
2023-04-caviar/src/PrivatePoolMetadata.sol::30 => return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));
```
