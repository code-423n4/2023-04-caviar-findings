# Protocol Overview

The Caviar Private Pools protocol is a decentralized platform on Ethereum that allows users to create and join private investment pools. The smart contract manages the funds within the pool, ensures compliance with the specified rules, and distributes rewards to members. The protocol democratizes investment opportunities, promotes decentralization, and fosters innovation in the investment industry.

| Total Low issues |
|------------------|

| Risk   | Issues Details                                                                                            | Number        |
|--------|-----------------------------------------------------------------------------------------------------------|---------------|
| [L-01] | Critical changes should use-two step procedure                                                            | 1             |
| [L-02] | No Storage Gap for Upgradeable contracts                                                                  | 1             |
| [L-03] | Loss of precision due to rounding                                                                         | 1             |
| [L-04] | Lack of `nonReentrant` modifier                                                                           | 4             |
| [L-05] | Integer overflow by unsafe casting                                                                        | 4             |
| [L-06] | Add `address(0)` check for the critical changes                                                           | 2             |
| [L-07] | Array lengths not checked                                                                                 | 4             |
| [L-08] | Unused `receive()` Function Will Lock Ether In Contract                                                   | 1             |
| [L-09] | Misleading comment                                                                                        | 1             |

| Total Non-Critical issues |
|---------------------------|

| Risk    | Issues Details                                                                                          | Number        |
|---------|---------------------------------------------------------------------------------------------------------|---------------|
| [NC-01] | Include return parameters in NatSpec comments                                                           | 6             |
| [NC-02] | Lack of event emit                                                                                      | 3             |
| [NC-03] | Mark visibility of `initialize()` functions as external                                                 | 1             |
| [NC-04] | Use `bytes.concat()` and `string.concat()`                                                              | 7             |
| [NC-05] | Use a more recent version of OpenZeppelin dependencies                                                  | 1             |
| [NC-06] | Constants in comparisons should appear on the left side                                                 | 8             |
| [NC-07] | `revert()` Statements Should Have Descriptive Reason Strings                                            | 1             |
| [NC-08] | Lock pragmas to specific compiler version                                                               | All Contracts |
| [NC-09] | There is no need to cast a variable that is already an address, such as `address(x)`                    | 2             |

| Total Stylistic issues |
|------------------------|

| Risk    | Issues Details                                                                                          | Number        |
|---------|---------------------------------------------------------------------------------------------------------|---------------|
| [S-01]  | Lines are too long                                                                                      | 5             |
| [S-02]  | Function writing does not comply with the `Solidity Style Guide`                                        | 2 Contracts   |
| [S-03]  | Generate perfect code headers every time                                                                | All Contracts |
| [S-04]  | For functions, follow Solidity standard naming conventions                                              | 1 Contracts   |

| Total Recommendations |
|-----------------------|

| Risk    | Issues Details                                                                                          | Number        |
|---------|---------------------------------------------------------------------------------------------------------|---------------|
| [R-01]  | Missing checks for `address(0)`                                                                         | 4             |
| [R-02]  | Contracts should have full test coverage                                                                |               |
| [R-03]  | Value is not validated to be different than the existing one                                            | 5             |
| [R-04]  | Add a timelock to critical functions                                                                    | 5             |
| [R-05]  | Need Fuzzing test                                                                                       | All Contracts |
| [R-06]  | Use SMTChecker                                                                                          |               |
| [R-07]  | Events that mark critical parameter changes should contain both the old and the new value               | 5             |

## [L-01] Critical changes should use-two step procedure

#### Description

The following contract [`Factory.sol`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol) inherit [`Owned.sol`](https://github.com/transmissions11/solmate/blob/main/src/auth/Owned.sol) which have a function that allows the owner to transfer ownership to a different address. If the owner accidentally uses an invalid address for which they do not have the private key, then the system will gets locked.

#### Lines of code 

```solidity
    function transferOwnership(address newOwner) public virtual onlyOwner {
        owner = newOwner;

        emit OwnershipTransferred(msg.sender, newOwner);
    }
```

- [Owned.sol#L39-L43](https://github.com/transmissions11/solmate/blob/main/src/auth/Owned.sol#L39-L43)

#### Recommended Mitigation Steps

Consider adding two step procedure on the critical functions where the first is announcing a pending new owner and the new address should then claim its ownership.

A similar issue was reported in a previous contest and was assigned a severity of medium: code-423n4/2021-06-realitycards-findings#105

## [L-02] No Storage Gap for Upgradeable contracts

#### Description

For upgradeable contracts, inheriting contracts may introduce new variables. In order to be able to add new variables to the upgradeable contract without causing storage collisions, a storage gap should be added to the upgradeable contract.

#### Lines of code 

- [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

#### Recommended Mitigation Steps

Consider adding a storage gap at the end of the upgradeable contract:

```solidity
  /**
   * @dev This empty reserved space is put in place to allow future versions to add new
   * variables without shifting down storage in the inheritance chain.
   * See https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps
   */
  uint256[50] private __gap;
```

## [L-03] Loss of precision due to rounding

#### Description

Loss of precision due to the nature of arithmetics and rounding errors.

#### Lines of code 

```solidity
        uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);
```

- [PrivatePool.sol#L719](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L719)

## [L-04] Lack of `nonReentrant` modifier

#### Description

It is a best practice to use the `nonReentrant` modifier when you make external calls to untrustable entities because even if an auditor did not think of a way to exploit it, an attacker just might.

#### Lines of code 

```solidity
    function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
```

- [PrivatePool.sol#L211](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211)

```solidity
    function sell(
```

- [PrivatePool.sol#L301](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301)

```solidity
    function change(
```

- [PrivatePool.sol#L385](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385)

```solidity
    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
```

- [PrivatePool.sol#L514](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514)

#### Recommended Mitigation Steps

Add reentrancy guard to the functions mentioned above.

## [L-05] Integer overflow by unsafe casting

#### Description

Keep in mind that the version of solidity used, despite being greater than 0.8, does not prevent integer overflows during casting, it only does so in mathematical operations.

It is necessary to safely convert between the different numeric types.

#### Lines of code 

```solidity
        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
```

- [PrivatePool.sol#L230](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230)

```solidity
        virtualNftReserves -= uint128(weightSum);
```

- [PrivatePool.sol#L231](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231)

```solidity
        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
```

- [PrivatePool.sol#L323](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323)

```solidity
        virtualNftReserves += uint128(weightSum);
```

- [PrivatePool.sol#L324](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324)

#### Recommended Mitigation Steps

Use OpenZeppelin [safeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) library.

## [L-06] Add `address(0)` check for the critical changes

#### Description

Check of `address(0)` to protect the code from `(0x0000000000000000000000000000000000000000)` address problem just in case. This is best practice or instead of suggesting that they verify `_address != address(0)`, you could add some good NatSpec comments explaining what is valid and what is invalid and what are the implications of accidentally using an invalid address.

#### Lines of code 

```solidity
    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
        privatePoolMetadata = _privatePoolMetadata;
    }
```

- [Factory.sol#L129-L131](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129-L131)

```solidity
    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }
```

- [Factory.sol#L135-L137](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135-L137)

#### Recommended Mitigation Steps

Add checks for `address(0)` when assigning values to address state variables.	

## [L-07] Array lengths not checked

#### Description

If the length of the arrays are not required to be of the same length, user operations may not be fully executed. Therefore, it's important to ensure that the arrays being operated on are properly aligned to avoid these issues.

#### Lines of code 

```solidity
    function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
```

- [PrivatePool.sol#L211](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211)

```solidity
    function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
```

- [PrivatePool.sol#L301-L306](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301-L306)

```solidity
    function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```

- [PrivatePool.sol#L385-L393](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385-L393)

```solidity
    function sumWeightsAndValidateProof(
        uint256[] memory tokenIds,
        uint256[] memory tokenWeights,
        MerkleMultiProof memory proof
    ) public view returns (uint256) {
```

- [PrivatePool.sol#L661-L665](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L661-L665)

#### Recommended Mitigation Steps

Check that the arrays length are the same.

## [L-08] Unused `receive()` Function Will Lock Ether In Contract

#### Description

[`EthRouter.sol`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol) has a payable` receive()` function for receiving ETH. But there is no way to withdraw it, thus the funds will be locked in the contract.

#### Lines of code 

```solidity
    receive() external payable {}
```

- [EthRouter.sol#L88](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88)

#### Recommended Mitigation Steps

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert, or add a function to rescue the locked Eth.

## [L-09] Misleading comment

#### Description

The current comment in the [`PrivatePool`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol) contract at line [376-377](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L376-L377) is misleading, as it states that the sum of the caller's NFT weights must be less than or equal to the sum of the output pool NFTs weights. However, the implementation of the contract does not reflect this comment, and it actually revert if the input weight sum is less than the output weight sum, which is the opposite of what the comment implies. This issue could potentially lead to confusion for developers and result in incorrect implementation of the contract.

#### Lines of code 

- [PrivatePool.sol#L376-L377](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L376-L377)

#### Recommended Mitigation Steps

To address this issue, we recommend changing the comment at line 376-377 to better reflect the intended functionality of the contract: 

```solidity
// Old comment: The sum of the caller's NFT weights must be less than or equal to the sum of the output pool NFTs weights.
// New comment: The sum of the caller's NFT weights must be greater than or equal to the sum of the output pool NFTs weights.
```

## [NC-01] Include return parameters in NatSpec comments

#### Description

If Return parameters are declared, you must prefix them with `/// @return`.
Some code analysis programs do analysis by reading [NatSpec](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html) details, if they can't see the `@return` tag, they do incomplete analysis.

#### Lines of code 

```solidity
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```

- [PrivatePool.sol#L393](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L393)

```solidity
    function flashFeeToken() public view returns (address) {
```

- [PrivatePool.sol#L755](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L755)

```solidity
    function tokenURI(uint256 tokenId) public view returns (string memory) {
```

- [PrivatePoolMetadata.sol#L17](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17)

```solidity
    function attributes(uint256 tokenId) public view returns (string memory) {
```

- [PrivatePoolMetadata.sol#L35](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L35)

```solidity
    function svg(uint256 tokenId) public view returns (bytes memory) {
```

- [PrivatePoolMetadata.sol#L55](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L55)

```solidity
    function trait(string memory traitType, string memory value) internal pure returns (string memory) {
```

- [PrivatePoolMetadata.sol#L112](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112)

#### Recommended Mitigation Steps

Include `@return` parameters in NatSpec comments

## [NC-02] Lack of event emit

#### Description

The below methods do not emit an event when the state changes, something that it's very important for dApps and users.

#### Lines of code 

```solidity
    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
        privatePoolMetadata = _privatePoolMetadata;
    }
```

- [Factory.sol#L129-L131](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129-L131)

```solidity
    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }
```

- [Factory.sol#L135-L137](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135-L137)

```solidity
    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }
```

- [Factory.sol#L141-L143](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141-L143)

#### Recommended Mitigation Steps

Emit event.

## [NC-03] Mark visibility of `initialize()` functions as external

#### Description

- If someone wants to extend via inheritance, it might make more sense that the overridden `initialize()` function calls the internal {...}_init function, not the parent public `initialize()` function.

- External instead of public would give more the sense of the `initialize()` functions to behave like a constructor (only called on deployment, so should only be called externally)

- Security point of view, it might be safer so that it cannot be called internally by accident in the child contract

- It might cost a bit less gas to use external over public

- It is possible to override a function from external to public ("opening it up") but it is not possible to override a function from public to external ("narrow it down").

> Ref: https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750

#### Lines of code 

```solidity
    function initialize(
        address _baseToken,
        address _nft,
        uint128 _virtualBaseTokenReserves,
        uint128 _virtualNftReserves,
        uint56 _changeFee,
        uint16 _feeRate,
        bytes32 _merkleRoot,
        bool _useStolenNftOracle,
        bool _payRoyalties
    ) public {
```

- [PrivatePool.sol#L157-L167](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157-L167)

#### Recommended Mitigation Steps

Change the visibility of `initialize()` functions to external.

## [NC-04] Use `bytes.concat()` and `string.concat()`

#### Description

Solidity version 0.8.4 introduces: 

- `bytes.concat()` vs `abi.encodePacked(<bytes>,<bytes>)`
- `string.concat()` vs `abi.encodePacked(<string>,<string>)`

https://docs.soliditylang.org/en/v0.8.17/types.html?highlight=bytes.concat#the-functions-bytes-concat-and-string-concat

#### Lines of code 

```solidity
        bytes memory metadata = abi.encodePacked(
```

- [PrivatePoolMetadata.sol#L19](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L19)

```solidity
        return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));
```

- [PrivatePoolMetadata.sol#L30](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L30)

```solidity
        bytes memory _attributes = abi.encodePacked(
```

- [PrivatePoolMetadata.sol#L39](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L39)

```solidity
            _svg = abi.encodePacked(
```

- [PrivatePoolMetadata.sol#L62](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L62)

```solidity
            _svg = abi.encodePacked(
```

- [PrivatePoolMetadata.sol#L81](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L81)

```solidity
            _svg = abi.encodePacked(
```

- [PrivatePoolMetadata.sol#L97](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L97)

```solidity
            abi.encodePacked(
```

- [PrivatePoolMetadata.sol#L115](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L115)

#### Recommended Mitigation Steps

Use `bytes.concat()` and `string.concat()`

## [NC-05] Use a more recent version of OpenZeppelin dependencies

#### Description

For security, it is best practice to use the latest OpenZeppelin dependencies version.

#### Lines of code 

```javaScript
  "dependencies": {
    "@openzeppelin/merkle-tree": "^1.0.2",
    "ethers": "^5.7.2"
  }
```

- [package.json#L16-L19](https://github.com/code-423n4/2023-04-caviar/blob/main/package.json#L16-L19)

#### Recommended Mitigation Steps

Old version of OpenZeppelin/merkle-tree is used `(^1.0.2)`, newer version can be used [`(1.0.4)`](https://www.npmjs.com/package/@openzeppelin/merkle-tree).

## [NC-06] Constants in comparisons should appear on the left side

#### Description

Constants in comparisons should appear on the left side, doing so will prevent typo [bug](https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html).

```solidity
        if (_baseToken == address(0)) {
```

#### Lines of code 

```solidity
        if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
```

- [Factory.sol#L87](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87)

```solidity
        if (_baseToken == address(0)) {
```

- [Factory.sol#L110](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L110)

```solidity
        if (token == address(0)) {
```

- [Factory.sol#L149](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L149)

```solidity
        if (baseToken == address(0)) {
```

- [PrivatePool.sol#L357](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L357)

```solidity
        if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
```

- [PrivatePool.sol#L489](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489)

```solidity
        if (token == address(0)) {
```

- [PrivatePool.sol#L522](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L522)

```solidity
        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
```

- [PrivatePool.sol#L635](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635)

```solidity
        if (merkleRoot == bytes32(0)) {
```

- [PrivatePool.sol#L667](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L667)

#### Recommended Mitigation Steps

Constants should appear on the left side:

```solidity
        if (address(0) == _baseToken) {
```

## [NC-07] `revert()` Statements Should Have Descriptive Reason Strings

#### Description

The correct and clear error description explains to the user why the function reverts, but the error descriptions below in the project are not self-explanatory. These error descriptions are very important in the debug features of DApps like Tenderly.

#### Lines of code 

```solidity
            revert();
```

- [PrivatePool.sol#L474](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L474)

#### Recommended Mitigation Steps

Error definitions should be added to the `revert("...")` block.

## [NC-08] Lock pragmas to specific compiler version

#### Description

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

```solidity
pragma solidity ^0.8.19;
```

> Ref: https://swcregistry.io/docs/SWC-103

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-04-caviar/tree/main/src)

#### Recommended Mitigation Steps

[Ethereum Smart Contract Best Practices](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/): Lock pragmas to specific compiler version. 

## [NC-09] There is no need to cast a variable that is already an address, such as `address(x)`

#### Description

There is no need to cast a variable that is already an `address`, such as `address(x)`, `x` is also `address`.

```solidity
            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
```

#### Lines of code 

```solidity
            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
```

- [PrivatePool.sol#L259](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L259)

```solidity
            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
```

- [PrivatePool.sol#L368](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L368)

#### Recommended Mitigation Steps     

Use directly variable:

```solidity
            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(factory, protocolFeeAmount);
```

## [S-01] Lines are too long

#### Description

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length.

> Ref: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

#### Lines of code 

```solidity
            trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))
```

- [PrivatePoolMetadata.sol#L47](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47)

```solidity
                '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 400 400" style="width:100%;background:black;fill:white;font-family:serif;">',
```

- [PrivatePoolMetadata.sol#L63](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L63)

```solidity
                        "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),
```

- [PrivatePoolMetadata.sol#L103](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103)

```solidity
    event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);
```

- [PrivatePool.sol#L58](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L58)

```solidity
    event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
```

- [PrivatePool.sol#L63](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L63)

## [S-02] Function writing does not comply with the `Solidity Style Guide`

#### Description

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

Functions should be grouped according to their visibility and ordered:

- `constructor()`
- `receive()`  
- `fallback()`  
- `external / public / internal / private`
- `view / pure`

#### Lines of code 

- [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

- [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

#### Recommended Mitigation Steps

Follow Solidity Style Guide.

## [S-03] Generate perfect code headers every time

#### Description

I recommend using header for Solidity code layout and readability

> Ref: https://github.com/transmissions11/headers

#### Lines of code 

- [All Contracts](https://github.com/code-423n4/2023-04-caviar/tree/main/src)

#### Recommended Mitigation Steps

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [S-04] For functions, follow Solidity standard naming conventions

#### Description

The protocol don't follow solidity standard naming convention.

> Ref: https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions

#### Lines of code 

- [PrivatePoolMetadata.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol)

#### Recommended Mitigation Steps

Follow solidity standard [naming convention](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#naming-conventions).

## [R-01] Missing checks for `address(0)`

#### Description

Check of the address for the critical changes to protect the code from address(0) problem in case of mistakes.

#### Lines of code 

```solidity
    constructor(address _royaltyRegistry) {
        royaltyRegistry = _royaltyRegistry;
    }
```

- [EthRouter.sol#L90-L92](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90-L92)

```solidity
    constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
        factory = payable(_factory);
        royaltyRegistry = _royaltyRegistry;
        stolenNftOracle = _stolenNftOracle;
    }
```

- [PrivatePool.sol#L143-L147](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143-L147)

#### Recommended Mitigation Steps

Add address(0) check.

## [R-02] Contracts should have full test coverage

#### Description

While 100% code coverage does not guarantee that there are no bugs, it often will catch easy-to-find bugs, and will ensure that there are fewer regressions when the code invariably has to be modified. Furthermore, in order to get full coverage, code authors will often have to re-organize their code so that it is more modular, so that each component can be tested separately, which reduces interdependencies between modules and layers, and makes for code that is easier to reason about and audit.

#### Recommended Mitigation Steps

Line coverage percentage should be 100%.

## [R-03] Value is not validated to be different than the existing one

#### Description

While the value is validated to be in the constant bounds, it is not validated to be different than the existing one. Queueing the same value will cause multiple abnormal events to be emitted, will ultimately result in a no-op situation.

#### Lines of code 

```solidity
    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
```

- [PrivatePool.sol#L538](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538)

```solidity
    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
```

- [PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550)

```solidity
    function setFeeRate(uint16 newFeeRate) public onlyOwner {
```

- [PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562)

```solidity
    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
```

- [PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576)

```solidity
    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```

- [PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587)

#### Recommended Mitigation Steps

Add a `require()` statement to check that the new value is different than the current one.

## [R-04] Add a timelock to critical functions

#### Description

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate.

#### Lines of code 

```solidity
    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }
```

- [Factory.sol#L141-L143](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141-L143)

```solidity
    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
        // set the virtual base token reserves and virtual nft reserves
        virtualBaseTokenReserves = newVirtualBaseTokenReserves;
        virtualNftReserves = newVirtualNftReserves;

        // emit the set virtual reserves event
        emit SetVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);
    }
```

- [PrivatePool.sol#L538-L545](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538-L545)

```solidity
    function setFeeRate(uint16 newFeeRate) public onlyOwner {
        // check that the fee rate is less than 50%
        if (newFeeRate > 5_000) revert FeeRateTooHigh();

        // set the fee rate
        feeRate = newFeeRate;

        // emit the set fee rate event
        emit SetFeeRate(newFeeRate);
    }
```

- [PrivatePool.sol#L562-L571](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562-L571)

```solidity
    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
        // set the use stolen NFT oracle flag
        useStolenNftOracle = newUseStolenNftOracle;

        // emit the set use stolen NFT oracle event
        emit SetUseStolenNftOracle(newUseStolenNftOracle);
    }
```

- [PrivatePool.sol#L576-L582](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576-L582)

```solidity
    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
        // set the pay royalties flag
        payRoyalties = newPayRoyalties;

        // emit the set pay royalties event
        emit SetPayRoyalties(newPayRoyalties);
    }
```

- [PrivatePool.sol#L587-L593](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587-L593)

#### Recommended Mitigation Steps

Consider adding a timelock to the critical changes.

## [R-05] Need Fuzzing test

#### Description

As Alberto Cuesta Canada said: 

> Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now.

Ref: https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05

#### Lines of code 

- [test](https://github.com/code-423n4/2023-04-caviar/tree/main/test)

#### Recommended Mitigation Steps

Use should fuzzing test like Echidna.

## [R-06] Use SMTChecker

#### Description

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

> Ref: https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19   

## [R-07] Events that mark critical parameter changes should contain both the old and the new value

#### Description

Events that mark critical parameter changes should contain both the old and the new value.

#### Lines of code

```solidity
        emit SetVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);
```

- [PrivatePool.sol#L544](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L544)

```solidity
        emit SetMerkleRoot(newMerkleRoot);
```

- [PrivatePool.sol#L555](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L555)

```solidity
        emit SetFeeRate(newFeeRate);
```

- [PrivatePool.sol#L570](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L570)

```solidity
        emit SetUseStolenNftOracle(newUseStolenNftOracle);
```

- [PrivatePool.sol#L581](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L581)

```solidity
        emit SetPayRoyalties(newPayRoyalties);
```

- [PrivatePool.sol#L592](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L592)

#### Recommended Mitigation Steps

Add the old value to the event.
