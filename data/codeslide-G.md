### Gas Optimizations

| Number | Issue | Instances |
| :----: | :---- | :-------: |
| [G-01] | Use short circuiting to save gas | 5 |
| [G-02] | Use assembly to check for address(0) | 22 |
| [G-03] | Change function visibility from public to external | 19 |
| [G-04] | State variables should be cached | 28 |
| [G-05] | Check amount before transfer | 8 |
| [G-06] | Mark functions as payable | 11 |
| [G-07] | Subtraction syntax | 2 |

#### [G-01] Use short circuiting to save gas
Use short circuiting to save gas by putting the cheaper comparison first.

```solidity
File: src/EthRouter.sol

// before - always reading block.timestamp
102:    if (block.timestamp > deadline && deadline != 0) {

// after - don't have to read block.timestamp if local variable is zero
102:    if (deadline != 0 && block.timestamp > deadline) {

// same as line 102
154:    if (block.timestamp > deadline && deadline != 0) {

// same as line 102
228:    if (block.timestamp > deadline && deadline != 0) {

// same as line 102
256:    if (block.timestamp > deadline && deadline != 0) {
```

#### [G-02] Use assembly to check for address(0)

Use assembly to check for address(0). Code example is at Solidity Assembly: Checking if an Address is 0 (Efficiently).

```solidity
File: src/Factory.sol

87:    if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

110:   if (_baseToken == address(0)) {
```

```solidity
File: src/PrivatePool.sol

225:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

254:    if (baseToken != address(0)) {

277:    if (royaltyFee > 0 && recipient != address(0)) {
278:        if (baseToken != address(0)) {

344:    if (royaltyFee > 0 && recipient != address(0)) {
345:        if (baseToken != address(0)) {

357:    if (baseToken == address(0)) {

397:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

421:    if (baseToken != address(0)) {

489:    if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

500:    if (baseToken != address(0)) {

522:    if (token == address(0)) {

635:    if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

651:    if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

733:    uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;

744:    uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
```

```solidity
File: src/PrivatePoolMetaData.sol

47:    trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))

103:    "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),
```

#### [G-03] Change function visibility from public to external
Change function visibility from `public` to `external` if the function is never called from within the contract. For public functions, the input parameters are copied to memory automatically, which costs gas. If the function is only called externally, then it should be marked as external since the parameters for an external function are not copied into memory but are read from calldata directly.

1. [EthRouter.sol L219](hhttps://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L219) `deposit()`
2. [EthRouter.sol L254](hhttps://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254) `change()`
3. [Factory.sol L71](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L71) `create()`
4. [Factory.sol L129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129) `setPrivatePoolMetadata()`
5. [Factory.sol L135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135) `setPrivatePoolImplementation()`
6. [Factory.sol L141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141) `setProtocolFeeRate()`
7. [Factory.sol L148](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148) `withdraw()`
8. [Factory.sol L161](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L161) `tokenURI()`
9. [Factory.sol L168](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L168) `predictPoolDeploymentAddress()`
10. [PrivatePool.sol L157](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157) `initialize()`
11. [PrivatePool.sol L211](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211) `buy()`
12. [PrivatePool.sol L301](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301) `sell()`
13. [PrivatePool.sol L385](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385) `change()`
14. [PrivatePool.sol L459](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459) `execute()`
15. [PrivatePool.sol L484](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484) `deposit()`
16. [PrivatePool.sol L514](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514) `withdraw()`
17. [PrivatePool.sol L602](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L602) `setAllParameters()`
18. [PrivatePool.sol L755](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L755) `flashFeeToken()`
19. [PrivatePoolMetadata.sol L17](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17) `tokenURI()`

#### [G-04] State variables should be cached

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

Saves 100 gas per instance

```solidity
File: src\PrivatePool.sol

// payRoyalties (second+ because it's in a for loop)
242:    if (payRoyalties) {

// baseToken
254:    if (baseToken != address(0)) {

// baseToken
256:    ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);

// baseToken
259:    if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

// factory
265:    if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

// payRoyalties
271:    if (payRoyalties) {
    
// baseToken
278:    if (baseToken != address(0)) {
279:        ERC20(baseToken).safeTransfer(recipient, royaltyFee);

// nft
331:    ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

// payRoyalties (second+ because it's in a for loop)
333:    if (payRoyalties) {
    
// baseToken
346:    ERC20(baseToken).safeTransfer(recipient, royaltyFee);

// baseToken
357:    if (baseToken == address(0)) {

// baseToken
365:    ERC20(baseToken).transfer(msg.sender, netOutputAmount);

// baseToken and factory
368:    if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

// baseToken
421:    if (baseToken != address(0)) {

// baseToken
423:    ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);

// baseToken
426:    if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);

// factory
432:    if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

// nft
442:    ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);

// nft
447:    ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);

// baseToken
489:    if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

// baseToken
500:    if (baseToken != address(0)) {

// baseToken
502:    ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);

// baseToken
651:    if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

// merkleRoot
682:    if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)) {

// baseToken
733:    uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;

// baseToken
744:    uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
```

#### [G-05] Check amount before transfer

Verify an amount is greater than zero before making an external call to transfer it. If the amount is zero, the transfer can be avoided and the gas needed for an external call can be saved.

```solidity
File: src\Factory.sol

112:    address(privatePool).safeTransferETH(baseTokenAmount);

115:    ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);

150:    msg.sender.safeTransferETH(amount);

152:    ERC20(token).transfer(msg.sender, amount);
```

```solidity
File: src\PrivatePool.sol

359:    msg.sender.safeTransferETH(netOutputAmount);

365:    ERC20(baseToken).transfer(msg.sender, netOutputAmount);

524:    msg.sender.safeTransferETH(tokenAmount);

527:    ERC20(token).transfer(msg.sender, tokenAmount);
```

#### [G-06] Mark functions as payable

Functions guaranteed to revert when called by normal users can be marked `payable`. If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

1. [Factory.sol L129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129) `setPrivatePoolMetadata()`
2. [Factory.sol L135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135) `setPrivatePoolImplementation()`
3. [Factory.sol L141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141) `setProtocolFeeRate()`
4. [Factory.sol L148](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148) `withdraw()`
5. [PrivatePool.sol L459](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459) `execute()`
6. [PrivatePool.sol L514](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514) `withdraw()`
7. [PrivatePool.sol L538](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538) `setVirtualReserves()`
8. [PrivatePool.sol L550](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550) `setMerkleRoot()`
9. [PrivatePool.sol L562](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562) `setFeeRate()`
10. [PrivatePool.sol L576](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576) `setUseStolenNftOracle()`
11. [PrivatePool.sol L587](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587) `setPayRoyalties()`

#### [G-07] Subtraction syntax

For state variables, x = x - y costs less gas than x -= y, and the same goes for addition. Consider replacing all -= and += occurrences to save gas.

```solidity
File: src\PrivatePool.sol

231:    virtualNftReserves -= uint128(weightSum);

323:    virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
```
