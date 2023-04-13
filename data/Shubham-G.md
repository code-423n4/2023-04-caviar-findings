## Gas Optimization

| |Issue|Instances|
|-|:-|:-:|
| [G-01](#G-01) | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables | 9 |
| [G-02](#G-02) | Setting the constructor to `payable` | 2 |

## [G-01] `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables

*There are 9 instances of this issue*

```solidity
File: main/src/PrivatePool.sol

L:230        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
L:231        virtualNftReserves -= uint128(weightSum);
L:247        royaltyFeeAmount += royaltyFee;
L:252        netInputAmount += royaltyFeeAmount;
L:323        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
L:324        virtualNftReserves += uint128(weightSum);
L:341        royaltyFeeAmount += royaltyFee;
L:355        netOutputAmount -= royaltyFeeAmount;
L:678        sum += tokenWeights[i];
```
## Recommended Step

```solidity
File: main/src/PrivatePool.sol

 - L:230        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
 + L:230        virtualBaseTokenReserves = virtualBaseTokenReserves + uint128(netInputAmount - feeAmount - protocolFeeAmount);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230

```solidity
File: main/src/PrivatePool.sol

 - L:231        virtualNftReserves -= uint128(weightSum);
 + L:231        virtualNftReserves = virtualNftReserves - uint128(weightSum);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231

```solidity
File: main/src/PrivatePool.sol

 - L:247        royaltyFeeAmount += royaltyFee;
 + L:247        royaltyFeeAmount = royaltyFeeAmount + royaltyFee;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L247

```solidity
File: main/src/PrivatePool.sol

 - L:252        netInputAmount += royaltyFeeAmount;
 + L:252        netInputAmount = netInputAmount + royaltyFeeAmount;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L252

```solidity
File: main/src/PrivatePool.sol

 - L:323        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
 + L:323        virtualBaseTokenReserves = virtualBaseTokenReserves - uint128(netOutputAmount + protocolFeeAmount + feeAmount);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323

```solidity
File: main/src/PrivatePool.sol

 - L:324        virtualNftReserves += uint128(weightSum);
 + L:324        virtualNftReserves = virtualNftReserves + uint128(weightSum);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324

```solidity
File: main/src/PrivatePool.sol

 - L:341        royaltyFeeAmount += royaltyFee;
 + L:341        royaltyFeeAmount = royaltyFeeAmount + royaltyFee;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L341

```solidity
File: main/src/PrivatePool.sol

 - L:355        netOutputAmount -= royaltyFeeAmount;
 + L:355        netOutputAmount = netOutputAmount - royaltyFeeAmount;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L355

```solidity
File: main/src/PrivatePool.sol

 - L:678        sum += tokenWeights[i];
 + L:678        sum = sum + tokenWeights[i];
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L678

**Gas Saved**

|Function name |Before|After|Diff (Avg)|
|-|:-|:-:|:-:|
| buy | 70844 | 70606 | 278 |
| deposit | 207588 | 29583 | 178005 |
| sell | 81969 | 81723 | 246 |

|Deployment Cost |Before|After|Diff|
|-|:-|:-:|:-:|
|  | 3248407 | 3233391 | 15016 |

## [GAS-02] Setting the constructor to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. The extra opcodes avoided are 
CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2). 
Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

```solidity
File: main/src/EthRouter.sol

- L: 90       constructor(address _royaltyRegistry) {
+ L: 90       constructor(address _royaltyRegistry) payable {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90

```solidity
File: main/src/PrivatePool.sol

- L:143      constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
+ L:143      constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) payable {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143

```solidity
File: main/src/Factory.sol

- L:53       constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
+ L:53       constructor() payable ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L53