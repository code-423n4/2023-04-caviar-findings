## Combining for loops in `buy()` of PrivatePool
Like it has been implemented in [`sell()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L328-L351), consider combining the two [for loops](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L242) involving [`if (payRoyalties)`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L271) in [`buy()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L289) to save gas both on function calls and contract size. 

## Identical arithmetic operation can be cached
In PrivatePool, the two identical arithmetic operations of `buy()` can be cached into a local variable to save gas on function call as follows:

[File: PrivatePool.sol#L229-L237](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L229-L237)

```diff
+        trimmedNetInputAmount = netInputAmount - feeAmount - protocolFeeAmount;

        // update the virtual reserves
-        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
+        virtualBaseTokenReserves += uint128(trimmedNetInputAmount);
        virtualNftReserves -= uint128(weightSum);

        // ~~~ Interactions ~~~ //

        // calculate the sale price (assume it's the same for each NFT even if weights differ)
-        uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
+        uint256 salePrice = trimmedNetInputAmount / tokenIds.length;
        uint256 royaltyFeeAmount = 0; 
```
## Unneeded `flashFeeToken()`
`baseToken` is already evidently used in [line 635](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635) and [line 651](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651) of `flashLoan()`. Additionally, `baseToken` comes with a free [public getter](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L82). Hence, consider removing the unneeded [`flashFeeToken()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L754-L757) to reduce the contract size.  

## Use of named returns for local variables saves gas
You can have further advantages in term of gas cost by simply using named return values as temporary local variable.

For instance, the code block below may be refactored as follows:

[File: PrivatePool.sol#L742-L746](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L742-L746)

```diff
-    function price() public view returns (uint256) {
+    function price() public view returns (uint256 _price) {
        // ensure that the exponent is always to 18 decimals of accuracy
        uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
        _price = (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;
    }
```
## += and -= cost more gas
`+=` and `-=` generally cost 22 more gas than writing out the assigned equation explicitly. The amount of gas wasted can be quite sizable when repeatedly operated in a loop.

For example, the `+=` instance below may be refactored as follows:

[File: PrivatePool.sol#L247](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L247)

```diff
-                royaltyFeeAmount += royaltyFee;
+                royaltyFeeAmount = royaltyFeeAmount + royaltyFee;
```
## `||` costs less gas than its equivalent `&&`
Rule of thumb: `(x && y)` is `(!(!x || !y))`

Even with the 10k Optimizer enabled: `||`, OR conditions cost less than their equivalent `&&`, AND conditions.

As an example, the instance below may be refactored as follows:

[File: EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101)

```diff
-        if (block.timestamp > deadline && deadline != 0) {
+        if (!(block.timestamp <= deadline || deadline == 0)) {
```
## Private function with embedded modifier reduces contract size
Consider having the logic of a modifier embedded through a private function to reduce contract size if need be. A `private` visibility that saves more gas on function calls than the `internal` visibility is adopted because the modifier will only be making this call inside the contract.

For example, the modifier instance below may be refactored as follows:

[File: PrivatePool.sol#L127-L132](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L127-L132)

```diff
+    function _onlyOwner() private view {
+        if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
+            revert Unauthorized();
+        }
+    }

    modifier onlyOwner() {
-        if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
-            revert Unauthorized();
-        }
+        _onlyOwner();
        _;
    }
```
## Payable access control functions costs less gas
Consider marking functions with access control as `payable`. This will save 20 gas on each call by their respective permissible callers for not needing to have the compiler check for `msg.value`.

For instance, the code line below may be refactored as follows:

[File: PrivatePool.sol#L514](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514)

```diff
-    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
+    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public payable onlyOwner {
```
## Function order affects gas consumption
The order of function will also have an impact on gas consumption. Because in smart contracts, there is a difference in the order of the functions. Each position will have an extra 22 gas. The order is dependent on method ID. So, if you rename the frequently accessed function to more early method ID, you can save gas cost. Please visit the following site for further information:

https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.17",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Please visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.