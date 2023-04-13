# Gas Optimizations Report

## [G-1] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact
Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings
Total:5

[src/PrivatePool.sol#L247-L678](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L247-L678)
```solidity
247:    royaltyFeeAmount += royaltyFee;

252:    netInputAmount += royaltyFeeAmount;

341:    royaltyFeeAmount += royaltyFee;

355:    netOutputAmount -= royaltyFeeAmount;

678:    sum += tokenWeights[i];
```

## [G-2] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

### Impact
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked). 

### Findings
Total:19


[src/EthRouter.sol#L284](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L284)
```solidity
106:    for (uint256 i = 0; i < buys.length; i++) {

116:    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

134:    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

159:    for (uint256 i = 0; i < sells.length; i++) {

161:    for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

183:    for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

239:    for (uint256 i = 0; i < tokenIds.length; i++) {

261:    for (uint256 i = 0; i < changes.length; i++) {

265:    for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {

```

[src/PrivatePool.sol#L238-L446](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L238-L446)
```solidity

238:    for (uint256 i = 0; i < tokenIds.length; i++) {

272:    for (uint256 i = 0; i < tokenIds.length; i++) {

329:    for (uint256 i = 0; i < tokenIds.length; i++) {

441:    for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:    for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:    for (uint256 i = 0; i < tokenIds.length; i++) {

518:    for (uint256 i = 0; i < tokenIds.length; i++) {

673:    for (uint256 i = 0; i < tokenIds.length; i++) {
```

[src/Factory.sol#L119](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol#L119)
```solidity
119:    for (uint256 i = 0; i < tokenIds.length; i++) {
```

## [G-3] Functions guaranteed to revert when called by normal users can be marked payable

### Impact
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
   The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings
Total:2

[src/Factory.sol#L129-L148](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol#L129-148)
```solidity
129:    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:    function withdraw(address token, uint256 amount) public onlyOwner {
```
[src/PrivatePool.sol#L459-L587](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L459-L587)
```solidity
459:    function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

514:    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538:    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550:    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562:    function setFeeRate(uint16 newFeeRate) public onlyOwner {

576:    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587:    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```

### Recommendation
Mark the function as payable.

## [G-4] Optimize names to save gas

### Impact
public/external function names and public member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call

### Findings
Total: 3

[src/PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol)
```solidity
       ///deposit,withdraw,setVirtualReserves,setMerkleRoot,setFeeRate...
45:    contract PrivatePool is ERC721TokenReceiver {
```

[src/Factory.sol](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol)
```solidity
       ///setPrivatePoolMetadata,setPrivatePoolImplementation,setProtocolFeeRate...
37:    contract Factory is ERC721, Owned {
```

[src/EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol)
```solidity
       ///deposit,getRoyalty
45:    contract EthRouter is ERC721TokenReceiver {
```