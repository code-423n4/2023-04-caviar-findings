## [G-1] Move checks to the top

Checks, effects, interactions is a general best practice and can be applicable to more than just reentrancy concerns. When one of the following error scenarios applies, users pay gas for all statements executed up until the revert itself. By performing checks such as these as early as possible, you are saving users gas in failure scenarios without any sacrifice to the happy case costs.

Moving the requirements to the top of the function can also improve readability.

There are 1 instances of this issue:

```solidity
File: /src/PrivatePool.sol

225:  if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L225

## [G-2] Use function for repetitive blocks of code

There is 1 instances of this issue:

```solidity
File: /src/PrivatePool.sol

277:    if (royaltyFee > 0 && recipient != address(0)) {
			if (baseToken != address(0)) {
				ERC20(baseToken).safeTransfer(recipient, royaltyFee);
			} else {
				recipient.safeTransferETH(royaltyFee);
			}
		}

```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277-L283


## [G-3] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

There are 11 instances of this issue:

```solidity
File: /src/Factory.sol

87:  if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L87

```solidity
File: /src/PrivatePool.sol

225:  if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

277:  if (royaltyFee > 0 && recipient != address(0)) {

344:  if (royaltyFee > 0 && recipient != address(0)) {

397:  if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

489:  if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

635:  if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L225

```solidity
File: /src/EthRouter.sol

101:  if (block.timestamp > deadline && deadline != 0) {

154:  if (block.timestamp > deadline && deadline != 0) {

228:  if (block.timestamp > deadline && deadline != 0) {

256:  if (block.timestamp > deadline && deadline != 0) {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L101

## [G-4] Use assembly to check for address(0)

Saves 6 gas per instance if using assembly to check for address(0)

There are 11 instances of this issue:

```solidity
File: /src/Factory.sol

87:  if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

110:  if (_baseToken == address(0)) {

149:  if (token == address(0)) {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L87

```solidity
File: /src/PrivatePool.sol

357:  if (baseToken == address(0)) {

489:  if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

522:  if (token == address(0)) {

635:  if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

733:  uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;

744:  uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L357

```solidity
File: /src/PrivatePoolMetadata.sol

47:  trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))

103:  "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L47

## [G-6] Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost

There are 12 instances of this issue:

```solidity
File: /src/Factory.sol

129:  function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:  function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:  function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:  function withdraw(address token, uint256 amount) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L129

```solidity
File: /src/PrivatePool.sol

127:  modifier onlyOwner() virtual {

459:  function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

514:  function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538:  function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550:  function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562:  function setFeeRate(uint16 newFeeRate) public onlyOwner {

576:  function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587:  function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L127

## [G-7] Use Custom Errors

[Source](https://blog.soliditylang.org/2021/04/21/custom-errors/)
Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors. This would save both deployment and runtime cost.

There are 6 instances of this issue:

```solidity
File: /src/PrivatePool.sol

221:  // calculate the required net input amount and fee amount

261:  // check that the caller sent enough ETH to cover the net required input

689:  /// @notice Returns the required input of buying a given amount of NFTs inclusive of the fee which is dependent on

692:  /// @return netInputAmount The required input amount of base tokens inclusive of the fee.

726:  /// @notice Returns the fee required to change a given amount of NFTs. The fee is based on the current changeFee

748:  /// @notice Returns the fee required to flash swap a given NFT.

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L221


