## Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [L-01](#L-01) | Update codes to avoid Compile Errors | 8 |
| [L-02](#L-02) | In the constructor, there is no return of incorrect address identification | 8 |

*Total 2 issues.*

## Non-Critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Lines too long | 4 |
| [NC-2](#NC-2) | Events are missing `indexed` field | 13 |
| [NC-3](#NC-3) | Missing NatSpec | 1 |
| [NC-4](#NC-4) | Spell check | 1 |
| [NC-5](#NC-5) | Floating pragma | 5 |
| [NC-6](#NC-6) | Assembly Codes Specific – Should Have Comments | 1 |
| [NC-7](#NC-7) | Pragma version^0.8.19 version too recent to be trusted | 5 |

*Total 7 issues.*

## [L-01] Update codes to avoid Compile Errors

*There are 8 instances of this issue.*

```solidity
warning[2519]: Warning: This declaration shadows an existing declaration.
  --> test/EthRouter/Change.t.sol:36:9:
   |
36 |         EthRouter.Change[] memory changes = new EthRouter.Change[](1);
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Note: The shadowed declaration is here:
 --> test/EthRouter/Change.t.sol:9:5:
  |
9 |     EthRouter.Change[] public changes;
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



warning[2519]: Warning: This declaration shadows an existing declaration.
  --> test/EthRouter/Change.t.sol:70:9:
   |
70 |         EthRouter.Change[] memory changes = new EthRouter.Change[](1);
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Note: The shadowed declaration is here:
 --> test/EthRouter/Change.t.sol:9:5:
  |
9 |     EthRouter.Change[] public changes;
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



warning[2519]: Warning: This declaration shadows an existing declaration.
   --> test/EthRouter/Change.t.sol:109:9:
    |
109 |         EthRouter.Change[] memory changes = new EthRouter.Change[](1);
    |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Note: The shadowed declaration is here:
 --> test/EthRouter/Change.t.sol:9:5:
  |
9 |     EthRouter.Change[] public changes;
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



warning[2072]: Warning: Unused local variable.
  --> test/Factory/Nft.t.sol:39:9:
   |
39 |         string memory tokenURI = factory.tokenURI(tokenId);
   |         ^^^^^^^^^^^^^^^^^^^^^^



warning[2072]: Warning: Unused local variable.
   --> test/PrivatePool/Quotes.t.sol:115:37:
    |
115 |         (uint256 returnedFeeAmount, uint256 protocolFeeAmount) = privatePool.changeFeeQuote(inputAmount);
    |                                     ^^^^^^^^^^^^^^^^^^^^^^^^^

```

## [L-02] In the constructor, there is no return of incorrect address identification

In case of incorrect address definition in the constructor , there is no way to fix it because of the variables are immutable.

```solidity
File: main/src/EthRouter.sol

L:90      constructor(address _royaltyRegistry) {
L:91         royaltyRegistry = _royaltyRegistry;
L:92      }
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90-L92

```solidity
File: main/src/PrivatePool.sol

L:143        constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
L:144            factory = payable(_factory);
L:145            royaltyRegistry = _royaltyRegistry;
L:146            stolenNftOracle = _stolenNftOracle;
L:147        }
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143-L147


## [NC-1] Lines too long

Lines in source code are typically limited to 80 characters, but it’s reasonable to stretch beyond this limit when need be as monitor screens theses days are comparatively larger. Considering the files will most likely reside in GitHub that will have a scroll bar automatically kick in when the length is over 164 characters, all code lines and comments should be split when/before hitting this length. Keep line width to max 120 characters for better readability where possible. Here are some of the instances entailed:
*There are 4 instances of this issue.*

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L58-L60
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L63


## [NC-2] Events are missing `indexed` field

Consider indexing parameters for events, serving as logs filter when looking for specifically wanted data. Up to three parameters in an event function can receive the attribute indexed which will cause the respective arguments to be treated as log topics instead of data.

```solidity
File:main/src/Factory.sol

L:41    event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);
L:42    event Withdraw(address indexed token, uint256 indexed amount);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L41-L42

```solidity
File: main/src/PrivatePool.sol

L:58    event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);
L:59    event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
L:60    event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
L:61    event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);
L:62    event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);
L:63    event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
L:64    event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);
L:65    event SetMerkleRoot(bytes32 merkleRoot);
L:66    event SetFeeRate(uint16 feeRate);
L:67    event SetUseStolenNftOracle(bool useStolenNftOracle);
L:68    event SetPayRoyalties(bool payRoyalties);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L58-L68

## [NC-3] Missing NatSpec

*There is 1 instance of this issue.*
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L166

## [NC-4] Spell check

*There is 1 instance of this issue.*

```solidity
File: main/src/PrivatePool.sol

L251:        // add the royalty fee amount to the net input aount
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L251

## [NC-5] Floating pragma

### Description
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List:

```solidity
File: main/src/EthRouter.sol

L:1        // SPDX-License-Identifier: MIT
L:2        pragma solidity ^0.8.19;

File: main/src/Factory.sol

L:1        // SPDX-License-Identifier: MIT
L:2        pragma solidity ^0.8.19;

File: main/src/PrivatePool.sol

L:1        // SPDX-License-Identifier: MIT
L:2        pragma solidity ^0.8.19;

File: main/src/PrivatePoolMetadata.sol

L:1        // SPDX-License-Identifier: MIT
L:2        pragma solidity ^0.8.19;

File: main/src/IStolenNftOracle.sol

L:1        // SPDX-License-Identifier: MIT
L:2        pragma solidity ^0.8.19;
```
## [NC-6] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```solidity
File: main/src/PrivatePool.sol

L:469            assembly {
L:470                let returnData_size := mload(returnData)
L:471                 revert(add(32, returnData), returnData_size)
L:472            }
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L469-L472

## [NC-7] Pragma version^0.8.19 version too recent to be trusted

https://github.com/ethereum/solidity/blob/develop/Changelog.md
0.8.19 (2023-02-22)
0.8.17 (2022-09-08)
0.8.16 (2022-08-08)
0.8.15 (2022-06-15)
0.8.10 (2021-11-09)

Unexpected bugs can be reported in recent versions;

Risks related to recent releases
Risks of complex code generation changes
Risks of new language features
Risks of known bugs
Use a non-legacy and more battle-tested version
Use 0.8.10