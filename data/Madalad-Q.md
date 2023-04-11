# Low Risk Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[L-01]|Use `safeTransfer` instead of `transfer` for token transfers|5|
|[L-02]|Downcasting `uint` or `int` may result in overflow|2|

Total issues: 2

Total instances: 7

&nbsp;
# Non-Critical Summary
| |Issue|Instances|
|:-:|:-|:-:|
|[N-01]|Use `indexed` for event parameters|11|
|[N-02]|Use fixed compiler version|4|
|[N-03]|Use appropriate function naming convention|1|
|[N-04]|Lines too long|4|
|[N-05]|Missing `address(0)` checks in constructor/initialize|3|

Total issues: 5

Total instances: 23

&nbsp;
# Low Risk Issues
## [L-01] Use `safeTransfer` instead of `transfer` for token transfers

`SafeERC20` is a wrapper around ERC20 operations that throw on failure (when the token contract returns false). This prevents calls executing if a token transfer is unsuccessful. Simply add `using SafeERC20 for ERC20`.

Instances: 5
```solidity
File: src/Factory.sol

115:             ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);

152:             ERC20(token).transfer(msg.sender, amount);

```
- [src/Factory.sol#L115](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L115)
- [src/Factory.sol#L152](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L152)

```solidity
File: src/PrivatePool.sol

365:             ERC20(baseToken).transfer(msg.sender, netOutputAmount);

527:             ERC20(token).transfer(msg.sender, tokenAmount);

651:         if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

```
- [src/PrivatePool.sol#L365](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L365)
- [src/PrivatePool.sol#L527](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L527)
- [src/PrivatePool.sol#L651](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651)

&nbsp;
## [L-02] Downcasting `uint` or `int` may result in overflow

Consider using OpenZeppelin's `SafeCast` library to prevent unexpected overflows.

Instances: 2
```solidity
File: src/PrivatePool.sol

231:         virtualNftReserves -= uint128(weightSum);

324:         virtualNftReserves += uint128(weightSum);

```
- [src/PrivatePool.sol#L231](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231)
- [src/PrivatePool.sol#L324](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324)

&nbsp;

&nbsp;

# Non-Critical Issues
## [N-01] Use `indexed` for event parameters

Index event fields make the field more quickly accessible to off-chain tools that parse events. 

If the variable is value type (uint, address, bool), then using `indexed` saves gas. Otherwise, each index field costs extra gas during emission.

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
## [N-02] Use fixed compiler version

A floating pragma risks a different compiler version being used in production vs testing, which poses security risks.

Instances: 4
```solidity
File: src/Factory.sol

2: pragma solidity ^0.8.19;

```
- [src/Factory.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L2)

```solidity
File: src/PrivatePoolMetadata.sol

2: pragma solidity ^0.8.19;

```
- [src/PrivatePoolMetadata.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L2)

```solidity
File: src/EthRouter.sol

2: pragma solidity ^0.8.19;

```
- [src/EthRouter.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L2)

```solidity
File: src/PrivatePool.sol

2: pragma solidity ^0.8.19;

```
- [src/PrivatePool.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L2)

&nbsp;
## [N-03] Use appropriate function naming convention

According to Solidity's style guide and popular convention, function names should be prefixed with an underscore if and only if they are `private` or `internal`.

See [here](https://primitivefi.notion.site/Solidity-Style-44daebebfbd645b0b9cbad7075ba42fe) for more details.


Instances: 1
```solidity
File: src/PrivatePoolMetadata.sol

112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

```
- [src/PrivatePoolMetadata.sol#L112](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112)

&nbsp;
## [N-04] Lines too long

[Solidity's style guide](https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length) states lines should be kept within 79 characters.
 
Also, if lines exceed 164 characters then a horizontal scroll bar will be required when viewing the file on Github.

Extensions such as prettier are a simple solution.

Instances: 4
```solidity
File: src/PrivatePoolMetadata.sol

47:             trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))

103:                         "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),

```
- [src/PrivatePoolMetadata.sol#L47](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47)
- [src/PrivatePoolMetadata.sol#L103](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103)

```solidity
File: src/PrivatePool.sol

58:     event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

63:     event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);

```
- [src/PrivatePool.sol#L58](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L58)
- [src/PrivatePool.sol#L63](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L63)

&nbsp;
## [N-05] Missing `address(0)` checks in constructor/initialize

Failing to check for invalid parameters on deployment may result in an erroneous input and require an expensive redeployment.

Instances: 3
```solidity
File: src/EthRouter.sol

90:     constructor(address _royaltyRegistry) {

```
- [src/EthRouter.sol#L90](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90)

```solidity
File: src/PrivatePool.sol

157:     function initialize(

143:     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {

```
- [src/PrivatePool.sol#L157](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157)
- [src/PrivatePool.sol#L143](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143)

&nbsp;
