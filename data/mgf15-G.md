
## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Using bools for storage incurs overhead | 3 |
| [GAS-2](#GAS-2) | Cache array length outside of loop | 19 |
| [GAS-3](#GAS-3) | Don't initialize variables with default value | 21 |
| [GAS-4](#GAS-4) | Functions guaranteed to revert when called by normal users can be marked `payable` | 10 |
| [GAS-5](#GAS-5) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 19 |
| [GAS-6](#GAS-6) | Use != 0 instead of > 0 for unsigned integer comparison | 17 |
### [GAS-1] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (3)*:
```solidity
File: 2023-04-caviar/src/PrivatePool.sol

94:     bool public initialized;

97:     bool public payRoyalties;

100:     bool public useStolenNftOracle;

```

### [GAS-2] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (19)*:
```solidity
File: 2023-04-caviar/src/EthRouter.sol

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
File: 2023-04-caviar/src/Factory.sol

119:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

238:         for (uint256 i = 0; i < tokenIds.length; i++) {

272:             for (uint256 i = 0; i < tokenIds.length; i++) {

329:         for (uint256 i = 0; i < tokenIds.length; i++) {

441:         for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:         for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:         for (uint256 i = 0; i < tokenIds.length; i++) {

518:         for (uint256 i = 0; i < tokenIds.length; i++) {

673:         for (uint256 i = 0; i < tokenIds.length; i++) {

```


### [GAS-3] Don't initialize variables with default value

*Instances (21)*:
```solidity
File: 2023-04-caviar/src/EthRouter.sol

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
File: 2023-04-caviar/src/Factory.sol

119:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

237:         uint256 royaltyFeeAmount = 0;

238:         for (uint256 i = 0; i < tokenIds.length; i++) {

272:             for (uint256 i = 0; i < tokenIds.length; i++) {

328:         uint256 royaltyFeeAmount = 0;

329:         for (uint256 i = 0; i < tokenIds.length; i++) {

441:         for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:         for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:         for (uint256 i = 0; i < tokenIds.length; i++) {

518:         for (uint256 i = 0; i < tokenIds.length; i++) {

673:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

### [GAS-4] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (10)*:
```solidity
File: 2023-04-caviar/src/Factory.sol

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:     function withdraw(address token, uint256 amount) public onlyOwner {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550:     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {

576:     function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587:     function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```

### [GAS-5] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (19)*:
```solidity
File: 2023-04-caviar/src/EthRouter.sol

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
File: 2023-04-caviar/src/Factory.sol

119:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

238:         for (uint256 i = 0; i < tokenIds.length; i++) {

272:             for (uint256 i = 0; i < tokenIds.length; i++) {

329:         for (uint256 i = 0; i < tokenIds.length; i++) {

441:         for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:         for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:         for (uint256 i = 0; i < tokenIds.length; i++) {

518:         for (uint256 i = 0; i < tokenIds.length; i++) {

673:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

### [GAS-6] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (17)*:
```solidity
File: 2023-04-caviar/src/EthRouter.sol

121:                         if (royaltyFee > 0) {

141:         if (address(this).balance > 0) {

188:                         if (royaltyFee > 0) {

290:         if (address(this).balance > 0) {

```

```solidity
File: 2023-04-caviar/src/Factory.sol

87:         if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

225:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

259:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

265:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

277:                 if (royaltyFee > 0 && recipient != address(0)) {

344:                 if (royaltyFee > 0 && recipient != address(0)) {

362:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

368:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

397:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

426:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);

432:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

467:         if (returnData.length > 0) {

489:         if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

```
