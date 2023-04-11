# Summary
|ID     | Optimization Details|  Gas saved |Instances|
|:----:  | :---           |  :----:         |:----:         |
| [G-01] | DUPLICATED REQUIRE()/IF() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION | 688 | 12 |
| [G-02] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO)| 1017 | 9 |
| [G-03] | EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING |  | 3 |
| [G-04] | OPTIMIZING EVENT PARAMETERS: INDEXING WHERE POSSIBLE | 1059 | 8 |
| [G-05] | USE DOBLE IF INSTEAD && | 4 | 7 |
| [G-06] | USE SHIFT RIGHT/LEFT INSTEAD OF DIVISION/MULTIPLICATION | 350 | 2 |
| [G-07] | USE NESTED IF AND, AVOID MULTIPLE CHECK COMBINATIONS | 550 | 11 |
| [G-08] | USE ASSEMBLY TO CHECK FOR ADDRESS(0) | 340 | 20 |

| Gas saved | 4004 | Total Instances | 72 |
|:--:|:--:|:--:|--:|

# Detailed Findings
## [G-01] DUPLICATED REQUIRE()/IF() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION 
```solidity
main/src/EthRouter.sol

// @audit Add a modifier

  101: if (block.timestamp > deadline && deadline != 0) {
  154: if (block.timestamp > deadline && deadline != 0) {
  228: if (block.timestamp > deadline && deadline != 0) {
  256: if (block.timestamp > deadline && deadline != 0) {
```
- [EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101) 
- [EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154)
- [EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228)
- [EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256)

```solidity
main/src/PrivatePool.sol

// @audit Add a modifier

  225: if (baseToken != address(0) && msg.value > 0)
  397: if (baseToken != address(0) && msg.value > 0)

  254: if (baseToken != address(0)) {
  278: if (baseToken != address(0)) {
  345: if (baseToken != address(0)) {
  421: if (baseToken != address(0)) {
  500: if (baseToken != address(0)) {
  651: if (baseToken != address(0))         
```
- [PrivatePool.sol#225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225) 
- [PrivatePool.sol#397](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L497) 
- [PrivatePool.sol#254](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L254) 
- [PrivatePool.sol#278](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L278) 
- [PrivatePool.sol#345](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345) 
- [PrivatePool.sol#421](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L421) 
- [PrivatePool.sol#500](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L500) 
- [PrivatePool.sol#651](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651) 

### RECOMMENDATION
One way to eliminate duplicated require() and if() checks in your Solidity code is to refactor them into a modifier. This can help to reduce code duplication and improve the readability and maintainability of your code.

To do this, you can create a modifier that includes the require() or if() check, and then apply the modifier wherever the check is needed in your code. This way, if you need to make changes to the check later on, you only need to do it in one place, rather than in multiple places throughout your code.

For example, you could create a modifier like this

```solidity
main/src/EthRouter.sol#L101

    modifier checkDeadline(uint256 deadline) {
        if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
        _;
    }
```
Here are the data available in the covered contracts. Taking advantage of this situation in non-covered contracts can also lead to gas consumption optimization.

## [G-02] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO)
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/catellaTech/8539f7345b17f0929a41598e7e00e3c2) in Solidity. The same applies to the subtraction operation.

```solidity
main/src/PrivatePool.sol

// @audit Refactorize

    230: virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
    231: virtualNftReserves -= uint128(weightSum);
    247: royaltyFeeAmount += royaltyFee;
    252: netInputAmount += royaltyFeeAmount;
    323: virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
    324: virtualNftReserves += uint128(weightSum);
    341: royaltyFeeAmount += royaltyFee;
    355: netOutputAmount -= royaltyFeeAmount;
    678: sum += tokenWeights[i];
```
### RECOMENDATION
```diff
main/src/PrivatePool.sol#L247
- 247: royaltyFeeAmount += royaltyFee;
+ 247: royaltyFeeAmount = royaltyFeeAmount + royaltyFee;
```

## [G-03] EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING
It is recommended to refactor the code to eliminate empty blocks, or to make the block perform some useful action, such as emitting an event or reverting. If the contract is meant to be extended, it should be declared as abstract and the function signatures should be added without any default implementation.

If the block is an empty if-statement block used to avoid performing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement. This is because the else-if/else conditions involve different classes of checks, which may lead to the introduction of errors when the code is later modified. For example, the structure if(x){}else if(y){...}else{...} should be replaced with if(!x){if(y){...}else{...}}.

Finally, empty receive()/fallback() functions with payable that are not used should be removed to save deployment gas on the blockchain.

```solidity
// @audit Refactorize
    main/src/Factory.sol:
    55: receive() external payable {}

    main/src/EthRouter.sol
    88: receive() external payable {}

    main/src/PrivatePool.sol
    134: receive() external payable {}
```
- [Factory.sol#L55](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L55)
- [EthRouter.sol#L88](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88)
- [PrivatePool.sol#L134](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L134)

## [G-04] OPTIMIZING EVENT PARAMETERS: INDEXING WHERE POSSIBLE
When creating events in Solidity, it is best practice to index the event parameters for efficient gas usage. The most gas-efficient approach is to index up to three event parameters. If there are fewer than three parameters, then it is recommended to index all of them. This will optimize the search and filtering functions of the Ethereum event logs. It is important to strike a balance between efficiency and functionality.

- [Factory.sol#L41](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L41)

```diff
-    event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);
+    event Create(address indexed privatePool, uint256[] indexed tokenIds, uint256 indexed baseTokenAmount);
```
The same optimization of adding an extra indexed parameter can be applied to:
- [PrivatePool.sol#L61](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L61)
- [PrivatePool.sol#L62](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L62)
- [PrivatePool.sol#L64](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L64)
- [PrivatePool.sol#L65](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L65)
- [PrivatePool.sol#L66](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L66)
- [PrivatePool.sol#L67](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L67)
- [PrivatePool.sol#L68](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L68)

## [G-05] USE DOBLE IF EN LUGAR DE &&
When the condition of the 'if' statement includes a logical 'AND' operator and is not followed by an 'else' statement, it is preferable to replace it with two separate 'if' statements. This not only makes the code more readable and easier to understand, but it can also be more optimal in terms of performance and resource consumption, as it avoids unnecessary evaluation of the second condition if the first one is false.

`main/src/Factory.sol`
- [Factory.sol#L87](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87)

`main/src/EthRouter.sol`
- [EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101)
- [EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154)
- [EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228)
- [EthRouter.sol#L234](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L234)
- [EthRouter.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256)

`main/src/PrivatePool.sol`
- [PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225)

## [G-06] USE SHIFT RIGHT/LEFT INSTEAD OF DIVISION/MULTIPLICATION
It is possible to calculate a division or multiplication by any power of 2 number x by shifting right or left, instead of using division or multiplication operations. This can be useful in terms of performance and gas consumption, as the shift operation SHR only uses 3 gas units, while the DIV operation uses 5 units.

In addition, Solidity's division operation includes a division-by-zero prevention that can further increase the cost of the operation. Using shifts instead of division can avoid this prevention and reduce gas consumption.

```diff
main/src/PrivatePool.sol:
- 703: protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() / 10_000;
+ 703: protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() >> 10_000;

- 704: feeAmount = inputAmount * feeRate / 10_000;
+ 703: feeAmount = inputAmount * feeRate >> 10_000;
```
- [PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L703-L704)
- [PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L721-L722)

## [G-07] USE NESTED IF AND, AVOID MULTIPLE CHECK COMBINATIONS
Using nested "if" statements instead of multiple "&&" check combinations can be cheaper in terms of gas consumption, but there are also additional advantages to consider. For example, nested "if" statements can make the code easier to read and understand, as well as provide better coverage reports. By breaking up complex logical checks into smaller, more easily digestible pieces, the code can be more maintainable and less error-prone over time. Additionally, using nested "if" statements can help avoid unexpected interactions between multiple conditions, leading to fewer bugs and more reliable code.

```solidity
main/src/EthRouter.sol
    87: if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
    101: if (block.timestamp > deadline && deadline != 0) {
    154: if (block.timestamp > deadline && deadline != 0) {
    228: if (block.timestamp > deadline && deadline != 0) {
    256: if (block.timestamp > deadline && deadline != 0) {

main/src/PrivatePool.sol
    225: if (baseToken != address(0) && msg.value > 0) 
    277: if (royaltyFee > 0 && recipient != address(0))
    344: if (royaltyFee > 0 && recipient != address(0))
    397: if (baseToken != address(0) && msg.value > 0) 
    489: if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0)))
    635: if (baseToken == address(0) && msg.value < fee)
```
### RECOMENDATION
```diff
main/src/EthRouter.sol#L101
- 101:    if (block.timestamp > deadline && deadline != 0) {
+         if (block.timestamp > deadline) {
+           if (deadline != 0) {
+           }
+         } 
```

## [G-08] USE ASSEMBLY TO CHECK FOR ADDRESS(0)
Saves 6 gas per instance if using assembly to check for address(0).

```solidity
main/src/Factory.sol
87:  if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) 
110: if (_baseToken == address(0)) 
149: if (token == address(0))

main/src/PrivatePoolMetadata.sol
47: trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) 
103: "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) 


main/src/PrivatePool.sol
    225: if (baseToken != address(0) && msg.value > 0) 
    154: if (baseToken != address(0))
    277: if (royaltyFee > 0 && recipient != address(0))
    278: if (baseToken != address(0))
    344: if (royaltyFee > 0 && recipient != address(0))
    345: if (baseToken != address(0))
    397: if (baseToken != address(0) && msg.value > 0) 
    421: if (baseToken != address(0))
    489: if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0)))
    522: if (token == address(0))
    635: if (baseToken == address(0) && msg.value < fee)
    733: uint256 exponent = baseToken == address(0) ? ...
    744: uint256 exponent = baseToken == address(0) ? ...
```
### RECOMENDATION
```solidity
// @audit example
assembly {
 if iszero(_addr) {
  mstore(0x00, "zero address")
  revert(0x00, 0x20)
 }
}
```
