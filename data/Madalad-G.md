# Gas Optimizations Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[G-01]|`abi.encodePacked` is more gas efficient than `abi.encode`|2|
|[G-02]|Functions guaranteed to revert when called by normal users can be marked `payable`|10|
|[G-03]|Use assembly to calculate hashes|2|
|[G-04]|Use `indexed` to save gas|11|
|[G-05]|Refactor modifiers to call a local function|1|
|[G-06]|Use `unchecked` for operations that cannot overflow/underflow|20|
|[G-07]|Change `public` functions to `external`|20|
|[G-08]|`x += y` costs more gas than `x = x + y` for state variables|4|
|[G-09]|Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead|5|
|[G-10]|Use named return values|12|

Total issues: 10

Total instances: 87

&nbsp;
# Gas Optimizations
## [G-01] `abi.encodePacked` is more gas efficient than `abi.encode`

`abi.encode` pads all elementary types to 32 bytes, whereas `abi.encodePacked` will only use the minimal required memory to encode the data. See [here](https://docs.soliditylang.org/en/v0.8.11/abi-spec.html?highlight=encodepacked#non-standard-packed-mode) for more info.

Instances: 2
```solidity
File: src/EthRouter.sol

177:                     abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))

```
- [src/EthRouter.sol#L177](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L177)

```solidity
File: src/PrivatePool.sol

675:             leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

```
- [src/PrivatePool.sol#L675](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L675)


&nbsp;
## [G-02] Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost (2400 per instance).

Instances: 10
```solidity
File: src/Factory.sol

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:     function withdraw(address token, uint256 amount) public onlyOwner {

```
- [src/Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129)
- [src/Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135)
- [src/Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141)
- [src/Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148)

```solidity
File: src/PrivatePool.sol

514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550:     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {

576:     function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587:     function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```
- [src/PrivatePool.sol#L514](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514)
- [src/PrivatePool.sol#L538](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538)
- [src/PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550)
- [src/PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562)
- [src/PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576)
- [src/PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587)


&nbsp;
## [G-03] Use assembly to calculate hashes

Saves 5000 deployment gas per instance and 80 runtime gas per instance.

### Unoptimized
```solidity
function solidityHash(uint256 a, uint256 b) public view {
	//unoptimized
	keccak256(abi.encodePacked(a, b));
}
```

### Optimized
```solidity
function assemblyHash(uint256 a, uint256 b) public view {
	//optimized
	assembly {
		mstore(0x00, a)
		mstore(0x20, b)
		let hashedVal := keccak256(0x00, 0x40)
	}
}
```

Instances: 2
```solidity
File: src/PrivatePool.sol

642:             receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");

675:             leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

```
- [src/PrivatePool.sol#L642](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L642)
- [src/PrivatePool.sol#L675](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L675)


&nbsp;
## [G-04] Use `indexed` to save gas
Using `indexed` for value type event parameters (address, uint, bool) saves at least 80 gas in emitting the event (https://gist.github.com/Tomosuke0930/9bf61e01a8c3e214d95a9b84dcb41d97).

Note that for other types however (string, bytes), it is more expensive.

Instances: 11
```solidity
File: src/Factory.sol

41:     event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);

```
- [src/Factory.sol#L41](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L41)

```solidity
File: src/PrivatePool.sol

58:     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

59:     event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);

60:     event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);

61:     event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);

62:     event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);

63:     event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);

64:     event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);

66:     event SetFeeRate(uint16 feeRate);

67:     event SetUseStolenNftOracle(bool useStolenNftOracle);

68:     event SetPayRoyalties(bool payRoyalties);

```
- [src/PrivatePool.sol#L58](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L58)
- [src/PrivatePool.sol#L59](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L59)
- [src/PrivatePool.sol#L60](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L60)
- [src/PrivatePool.sol#L61](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L61)
- [src/PrivatePool.sol#L62](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L62)
- [src/PrivatePool.sol#L63](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L63)
- [src/PrivatePool.sol#L64](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L64)
- [src/PrivatePool.sol#L66](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L66)
- [src/PrivatePool.sol#L67](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L67)
- [src/PrivatePool.sol#L68](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L68)


&nbsp;
## [G-05] Refactor modifiers to call a local function

Modifiers code is copied in all instances where it's used, increasing bytecode size. By doing a refractor to the internal function, one can reduce bytecode size significantly at the cost of one JUMP.

Instances: 1
```solidity
File: src/PrivatePool.sol

127:     modifier onlyOwner() virtual {

```
- [src/PrivatePool.sol#L127](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L127)


&nbsp;
## [G-06] Use `unchecked` for operations that cannot overflow/underflow

By bypassing Solidity's built in overflow/underflow checks using `unchecked`, we can save gas. This is especially beneficial for the index variable within for loops (saves 120 gas per iteration).

Instances: 20
```solidity
File: src/Factory.sol

119:         for (uint256 i = 0; i < tokenIds.length; i++) {

```
- [src/Factory.sol#L119](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L119)

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
- [src/EthRouter.sol#L106](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L106)
- [src/EthRouter.sol#L116](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L116)
- [src/EthRouter.sol#L134](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L134)
- [src/EthRouter.sol#L159](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L159)
- [src/EthRouter.sol#L161](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L161)
- [src/EthRouter.sol#L183](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L183)
- [src/EthRouter.sol#L239](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L239)
- [src/EthRouter.sol#L261](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L261)
- [src/EthRouter.sol#L265](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L265)
- [src/EthRouter.sol#L284](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L284)

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

678:             sum += tokenWeights[i];

```
- [src/PrivatePool.sol#L238](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238)
- [src/PrivatePool.sol#L272](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L272)
- [src/PrivatePool.sol#L329](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329)
- [src/PrivatePool.sol#L441](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441)
- [src/PrivatePool.sol#L446](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L446)
- [src/PrivatePool.sol#L496](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L496)
- [src/PrivatePool.sol#L518](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L518)
- [src/PrivatePool.sol#L673](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L673)
- [src/PrivatePool.sol#L678](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L678)


&nbsp;
## [G-07] Change `public` functions to `external`

Functions marked as `public` that are not called internally should be set to `external` to save gas and improve code quality. External call cost is less expensive than of public functions.

Instances: 20
```solidity
File: src/Factory.sol

84:     ) public payable returns (PrivatePool privatePool) {

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:     function withdraw(address token, uint256 amount) public onlyOwner {

168:     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

```
- [src/Factory.sol#L84](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L84)
- [src/Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129)
- [src/Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135)
- [src/Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141)
- [src/Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148)
- [src/Factory.sol#L168](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L168)

```solidity
File: src/PrivatePoolMetadata.sol

17:     function tokenURI(uint256 tokenId) public view returns (string memory) {

```
- [src/PrivatePoolMetadata.sol#L17](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17)

```solidity
File: src/EthRouter.sol

99:     function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {

226:     ) public payable {

254:     function change(Change[] calldata changes, uint256 deadline) public payable {

```
- [src/EthRouter.sol#L99](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L99)
- [src/EthRouter.sol#L226](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L226)
- [src/EthRouter.sol#L254](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254)

```solidity
File: src/PrivatePool.sol

167:     ) public {

212:         public

306:     ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {

393:     ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

484:     function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {

514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

609:     ) public {

742:     function price() public view returns (uint256) {

755:     function flashFeeToken() public view returns (address) {

```
- [src/PrivatePool.sol#L167](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L167)
- [src/PrivatePool.sol#L212](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L212)
- [src/PrivatePool.sol#L306](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L306)
- [src/PrivatePool.sol#L393](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L393)
- [src/PrivatePool.sol#L459](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459)
- [src/PrivatePool.sol#L484](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484)
- [src/PrivatePool.sol#L514](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514)
- [src/PrivatePool.sol#L609](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L609)
- [src/PrivatePool.sol#L742](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L742)
- [src/PrivatePool.sol#L755](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L755)


&nbsp;
## [G-08]  `x += y` costs more gas than `x = x + y` for state variables

Instances: 4
```solidity
File: src/PrivatePool.sol

230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231:         virtualNftReserves -= uint128(weightSum);

323:         virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324:         virtualNftReserves += uint128(weightSum);

```
- [src/PrivatePool.sol#L230](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230)
- [src/PrivatePool.sol#L231](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231)
- [src/PrivatePool.sol#L323](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323)
- [src/PrivatePool.sol#L324](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324)


&nbsp;
## [G-09] Usage of `uint` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

Consider using a larger size then downcasting where needed.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

Instances: 5
```solidity
File: src/Factory.sol

51:     uint16 public protocolFeeRate;

```
- [src/Factory.sol#L51](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L51)

```solidity
File: src/PrivatePool.sol

88:     uint56 public changeFee;

91:     uint16 public feeRate;

104:     uint128 public virtualBaseTokenReserves;

112:     uint128 public virtualNftReserves;

```
- [src/PrivatePool.sol#L88](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L88)
- [src/PrivatePool.sol#L91](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L91)
- [src/PrivatePool.sol#L104](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L104)
- [src/PrivatePool.sol#L112](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L112)


&nbsp;
## [G-10] Use named return values

Using named return values instead of explicitly calling `return` saves ~13 execution gas per call and >1000 deployment gas per instance.

Instances: 12
```solidity
File: src/Factory.sol

161:     function tokenURI(uint256 id) public view override returns (string memory) {

```
- [src/Factory.sol#L161](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L161)

```solidity
File: src/PrivatePoolMetadata.sol

17:     function tokenURI(uint256 tokenId) public view returns (string memory) {

35:     function attributes(uint256 tokenId) public view returns (string memory) {

55:     function svg(uint256 tokenId) public view returns (bytes memory) {

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

```
- [src/PrivatePoolMetadata.sol#L17](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17)
- [src/PrivatePoolMetadata.sol#L35](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L35)
- [src/PrivatePoolMetadata.sol#L55](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L55)
- [src/PrivatePoolMetadata.sol#L112](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112)

```solidity
File: src/PrivatePool.sol

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

626:         returns (bool)

665:     ) public view returns (uint256) {

742:     function price() public view returns (uint256) {

750:     function flashFee(address, uint256) public view returns (uint256) {

755:     function flashFeeToken() public view returns (address) {

763:     function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {

```
- [src/PrivatePool.sol#L459](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459)
- [src/PrivatePool.sol#L626](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L626)
- [src/PrivatePool.sol#L665](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L665)
- [src/PrivatePool.sol#L742](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L742)
- [src/PrivatePool.sol#L750](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L750)
- [src/PrivatePool.sol#L755](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L755)
- [src/PrivatePool.sol#L763](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L763)


&nbsp;
