## Summary 

### Gas Optimization

|no |Issue|Instances|
|----|-------|------|
| [G&#x2023;1] | Use double if statements instead of && | 10 | - |
| [G&#x2023;2] | Make 3 event parameters indexed when possible| 2 | - |
| [G&#x2023;3] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES| 7 | - |
| [G&#x2023;4] | ++variable costs less gas than variable++, especially when it’s used in for-loops (—variable/variable— too)| 19 | - |
| [G&#x2023;5] | public functions to external| 1 | - |
| [G&#x2023;6] | Use selfbalance() instead of address(this).balance| 6 | - |
| [G&#x2023;7] | Remove the initializer modifier use require  | 1 | - |


## Gas Optimizations  

### [G-1] Use double if statements instead of &&
```solidity
file:    Factory.sol
78       if ((_baseToken == address(0) && msg.value !=  baseTokenAmount) || (_baseToken != address(0) && msg.value > 0))

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87

```solidity
file: EthRouter.sol
101   if (block.timestamp > deadline && deadline != 0)
154   if (block.timestamp > deadline && deadline != 0)
256   if (block.timestamp > deadline && deadline != 0)
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256

```solidity
file:  PrivatePool.sol
225    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
277    if (royaltyFee > 0 && recipient != address(0))
344    if (royaltyFee > 0 && recipient != address(0))
397    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
489    if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0)))
635    if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();


```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635


### [G-2]  Make 3 event parameters indexed when possible

```solidity
file    Factory.sol
41      event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);
42      event Withdraw(address indexed token, uint256 indexed amount);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L41
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L42

### [G-3] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES
```solidity
file:    PrivatePool.sol
231      virtualNftReserves -= uint128(weightSum);
247      royaltyFeeAmount += royaltyFee;
252      netInputAmount += royaltyFeeAmount;
324      virtualNftReserves += uint128(weightSum);
341      royaltyFeeAmount += royaltyFee;
355      netOutputAmount -= royaltyFeeAmount;
678      sum += tokenWeights[i];

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L247
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L252
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L341
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L355
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L678

### [G-4] ++variable costs less gas than variable++, especially when it’s used in for-loops (—variable/variable— too)
```solidity
file:    Factory.sol
119      for (uint256 i = 0; i < tokenIds.length; i++) 
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L119

```solidity 
file:    EthRouter.sol
106      for (uint256 i = 0; i < buys.length; i++)
116      for (uint256 j = 0; j < buys[i].tokenIds.length; j++)
134      for (uint256 j = 0; j < buys[i].tokenIds.length; j++)
159      for (uint256 i = 0; i < sells.length; i++)
161      for (uint256 j = 0; j < sells[i].tokenIds.length; j++) 
183      for (uint256 j = 0; j < sells[i].tokenIds.length; j++)
239      for (uint256 i = 0; i < tokenIds.length; i++)
261      for (uint256 i = 0; i < changes.length; i++)
265      for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++)
284      for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) 

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L106
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L116
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L134
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L159
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L161
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L183
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L239
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L261
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L265
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L284

```solidity
file:    PrivatePool.sol
238      for (uint256 i = 0; i < tokenIds.length; i++)
272      for (uint256 i = 0; i < tokenIds.length; i++)
329      for (uint256 i = 0; i < tokenIds.length; i++)
441      for (uint256 i = 0; i < inputTokenIds.length; i++)
446      for (uint256 i = 0; i < outputTokenIds.length; i++)  
496      for (uint256 i = 0; i < tokenIds.length; i++)
518      for (uint256 i = 0; i < tokenIds.length; i++)
673      for (uint256 i = 0; i < tokenIds.length; i++) 

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L272
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L446
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L496
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L518
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L673


### [G-5]  public functions to external
```solidity
file:   Factory.sol
161     function tokenURI(uint256 id) public view override returns (string memory)
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L161

### [G-6]  Use selfbalance() instead of address(this).balance
```solidity
file:   EthRouter.sol
141     if (address(this).balance > 0)
142     msg.sender.safeTransferETH(address(this).balance);
203     if (address(this).balance < minOutputAmount) 
208     msg.sender.safeTransferETH(address(this).balance);
290     if (address(this).balance > 0)
291     msg.sender.safeTransferETH(address(this).balance);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L141
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L142
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L203
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L208
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L290
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L291

### [G-7] Remove the initializer modifier use require.
```solidity
file:   PrivatePool.sol
157     function initialize

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157
