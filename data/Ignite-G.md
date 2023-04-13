
### Gas Optimizations
| |Issue|Instances|Average Gas Saved|
|---|---|---|---|
| Gas-1 | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables | 4 | - |
| Gas-2 | Setting the constructor to payable | 2 | 26 |
| Gas-3 | Optimize function names to save gas | All Contracts |- |
| Gas-4 | Using a ternary operator instead of an "if-else" statement | - |
| Gas-5 | Use assembly to check for `address(0)` | 23 | 138 |
| Gas-6 | Using private rather than public for `immutable`, saves gas | 4 | - |
 
### [Gas-1] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (`<x> -= <y>` too) 
Using the addition operator instead of plus-equals saves gas.

#### <ins>Instances(4):</ins>

```solidity
230: virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231: virtualNftReserves -= uint128(weightSum);

323: virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324: virtualNftReserves += uint128(weightSum);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324

#### <ins>Recommendation</ins>
Using `<x> = <x> + <y>` instead.

### [Gas-2] Setting the constructor to payable

Saves 13 gas per instance.

#### <ins>Instances(2):</ins>

```solidity
53: constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143

```solidity
90: constructor(address _royaltyRegistry) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90

```solidity
143: constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L53


#### <ins>Recommendation</ins>
Setting the constructor to payable.

### [Gas-3] Optimize function names to save gas

Optimizing function names can save gas because lower method ID will result in smaller gas, which can be significant for contracts with a large number of function calls.

Reference: https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

#### <ins>Instances:</ins>

All in-scope contracts

#### <ins>Recommendation</ins>
Find a lower method ID name for the most called functions.

### [Gas-4] Using a ternary operator instead of an "if-else" statement

For simple "if-else" statements that are easy to read, replacing a "if-else" statement with a ternary operator can help you save gas.

#### <ins>Instances(5):</ins>
```solidity
110: if (_baseToken == address(0)) {
    
149: if (token == address(0)) {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L110

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L149

```solidity
278: if (baseToken != address(0)) {

345: if (baseToken != address(0)) {

522: if (token == address(0)) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L278

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L522

#### <ins>Recommendation</ins>
Using ternary operator for simple "if-else" statements.

### [GAS-5] Use assembly to check for `address(0)`
Saves 6 gas per instance.

#### <ins>Instances(23):</ins>

```solidity
87: if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
    
110: if (_baseToken == address(0)) {

149: if (token == address(0)) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L110

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L149

```solidity
225: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

254: if (baseToken != address(0)) {

277: if (royaltyFee > 0 && recipient != address(0)) {

278: if (baseToken != address(0)) {
    
344: if (royaltyFee > 0 && recipient != address(0)) {

345: if (baseToken != address(0)) {
    
357: if (baseToken == address(0)) {

397: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

421: if (baseToken != address(0)) {

489: if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

500: if (baseToken != address(0)) {

522: if (token == address(0)) {

635: if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

651: if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

733: uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;

744: uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L254

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L278

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L278

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L357

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L421

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L500

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L522

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L733

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L744

```solidity
47: trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))

103: "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103

#### <ins>Recommendation</ins>
Use assembly to check for `address(0)`.

### [GAS-6] Using private rather than public for `immutable`, saves gas

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

#### <ins>Instances(4):</ins>

```solidity
86: address public immutable royaltyRegistry;
```    

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L86

```solidity
119: address public immutable stolenNftOracle;

122: address payable public immutable factory;

125: address public immutable royaltyRegistry;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L119

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L122

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L125

#### <ins>Recommendation</ins>
Using private rather than public for `immutable` state.