# Summary
A majority of the optimizations were benchmarked via the protocol's tests, i.e. using the following config: `solc version 0.8.19`, `optimizer on`, and `200 runs`. Optimizations that were not benchmarked are explained via EVM gas costs and opcodes.

Below are the overall average gas savings for the following tested functions (with all the optimizations applied):
| Function |    Before   |    After   | Avg Gas Savings |
| ------ | -------- | -------- | ------- |
| EthRouter.buy |  199750  |  190464  |  9286 | 
| EthRouter.change |  217295  |  202568  |  14727 |  
| EthRouter.deposit |  29900  |  29393  | 507 | 
| EthRouter.sell |  232102  |  223981  | 8121 | 
| Factory.create |  148801  |  148672 | 129 | 
| PrivatePool.buy |  70884  |  69821  | 1063 | 
| PrivatePool.change |  82138  |  78083  | 4055 | 
| PrivatePool.execute |  18890  |  18550  | 340 | 
| PrivatePool.flashLoan |  83063  |  82915  | 148 | 
| PrivatePool.sell |  81969  |  81284  | 685 | 
| PrivatePool.withdraw |  62023  |  61038  | 985 | 

**Total gas saved across all listed functions: 40046**

*Notes*: 

- The [Gas report](#gasreport-output-with-all-optimizations-applied) output, after all optimizations have been applied, can be found at the end of the report.
- The final diffs for each contract, with all the optimizations applied, can be found [here](https://gist.github.com/0xJCN/b03b5f1f8cabc937c8bbe4d4a46b8d47).
- Some code snippets may be truncated to save space. Code snippets may also be accompanied by `@audit` tags in comments to aid in explaining the issue.

## Gas Optimizations
| Number |Issue|Instances|
|-|:-|:-:| 
| [G-01](#cache-calldatamemory-pointers-for-complex-types-to-avoid-offset-calculations) | Cache calldata/memory pointers for complex types to avoid offset calculations | 52 | 
| [G-02](#use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated) | Use calldata instead of memory for function arguments that do not get mutated | 4 | 
| [G-03](#state-variables-can-be-cached-instead-of-re-reading-them-from-storage) | State variables can be cached instead of re-reading them from storage | 16 |
| [G-04](#cache-state-variables-outside-of-loop-to-avoid-reading-storage-on-every-iteration) | Cache state variables outside of loop to avoid reading storage on every iteration | 6 |
| [G-05](#rearrange-code-to-fail-early) | Rearrange code to fail early | 1 |
| [G-06](#x--yx---y-costs-more-gas-than-x--x--yx--x---y-for-state-variables) | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables | 2 | 
| [G-07](#if-statements-that-use--can-be-refactored-into-nested-if-statements) | `If` statements that use `&&` can be refactored into nested `if` statements | 9 |
| [G-09](#use-assembly-for-loops) | Use assembly for loops | 11 | 

## Cache calldata/memory pointers for complex types to avoid offset calculations
The function parameters in the following instances are complex types (arrays of structs which contain arrays) and thus will result in more complex offset calculations to retrieve specific data from calldata/memory. We can avoid peforming some of these offset calculations by instantiating calldata/memory pointers.

Total Instances: `52`

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L106-L138

### Cache calldata pointers for `buys[i]` and `buys[i].tokenIds`

*Gas Savings for `EthRouter.buy`, obtained via protocol's tests: Avg 5987 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  187054  |  397581  |  199750 |    7     |
| After  |  182524  |  381578  |  193763 |    7     |

```solidity
File: src/EthRouter.sol
106:        for (uint256 i = 0; i < buys.length; i++) {
107:            if (buys[i].isPublicPool) {
108:                // execute the buy against a public pool
109:                uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(
110:                    buys[i].tokenIds, buys[i].baseTokenAmount, 0
111:                );
112:
113:                // pay the royalties if buyer has opted-in
114:                if (payRoyalties) {
115:                    uint256 salePrice = inputAmount / buys[i].tokenIds.length;
116:                    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
117:                        // get the royalty fee and recipient
118:                        (uint256 royaltyFee, address royaltyRecipient) =
119:                            getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);
120:
121:                        if (royaltyFee > 0) {
122:                            // transfer the royalty fee to the royalty recipient
123:                            royaltyRecipient.safeTransferETH(royaltyFee);
124:                        }
125:                    }
126:                }
127:            } else {
128:                // execute the buy against a private pool
129:                PrivatePool(buys[i].pool).buy{value: buys[i].baseTokenAmount}(
130:                    buys[i].tokenIds, buys[i].tokenWeights, buys[i].proof
131:                );
132:            }
133:
134:            for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
135:                // transfer the NFT to the caller
136:                ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
137:            }
138:        }
```
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..1ed1c17 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -104,19 +104,21 @@ contract EthRouter is ERC721TokenReceiver {

         // loop through and execute the the buys
         for (uint256 i = 0; i < buys.length; i++) {
-            if (buys[i].isPublicPool) {
+            Buy calldata _buy = buys[i];
+            uint256[] calldata _tokenIds = _buy.tokenIds;
+            if (_buy.isPublicPool) {
                 // execute the buy against a public pool
-                uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(
-                    buys[i].tokenIds, buys[i].baseTokenAmount, 0
+                uint256 inputAmount = Pair(_buy.pool).nftBuy{value: _buy.baseTokenAmount}(
+                    _tokenIds, _buy.baseTokenAmount, 0
                 );

                 // pay the royalties if buyer has opted-in
                 if (payRoyalties) {
-                    uint256 salePrice = inputAmount / buys[i].tokenIds.length;
-                    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
+                    uint256 salePrice = inputAmount / _tokenIds.length;
+                    for (uint256 j = 0; j < _tokenIds.length; j++) {
                         // get the royalty fee and recipient
                         (uint256 royaltyFee, address royaltyRecipient) =
-                            getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);
+                            getRoyalty(_buy.nft, _tokenIds[j], salePrice);

                         if (royaltyFee > 0) {
                             // transfer the royalty fee to the royalty recipient
@@ -126,14 +128,14 @@ contract EthRouter is ERC721TokenReceiver {
                 }
             } else {
                 // execute the buy against a private pool
-                PrivatePool(buys[i].pool).buy{value: buys[i].baseTokenAmount}(
-                    buys[i].tokenIds, buys[i].tokenWeights, buys[i].proof
+                PrivatePool(_buy.pool).buy{value: _buy.baseTokenAmount}(
+                    _tokenIds, _buy.tokenWeights, _buy.proof
                 );
             }

-            for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
+            for (uint256 j = 0; j < _tokenIds.length; j++) {
                 // transfer the NFT to the caller
-                ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
+                ERC721(_buy.nft).safeTransferFrom(address(this), msg.sender, _tokenIds[j]);
             }
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L159-L200

### Cache calldata pointers for `sells[i]` and `sells[i].tokenIds`

*Gas Savings for `EthRouter.sell`, obtained via protocol's tests: Avg 5422 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  217300  |  402940  |  232102 |    7     |
| After  |  212264  |  392556  |  226680 |    7     |

```solidity
File: src/EthRouter.sol
159:        for (uint256 i = 0; i < sells.length; i++) {
160:            // transfer the NFTs into the router from the caller
161:            for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
162:                ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
163:            }
164:
165:            // approve the pair to transfer NFTs from the router
166:            ERC721(sells[i].nft).setApprovalForAll(sells[i].pool, true);
167:
168:            if (sells[i].isPublicPool) {
169:                // exceute the sell against a public pool
170:                uint256 outputAmount = Pair(sells[i].pool).nftSell(
171:                    sells[i].tokenIds,
172:                    0,
173:                    0,
174:                    sells[i].publicPoolProofs,
175:                    // ReservoirOracle.Message[] is the exact same as IStolenNftOracle.Message[] and can be
176:                    // decoded/encoded 1-to-1.
177:                    abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))
178:                );
179:
180:                // pay the royalties if seller has opted-in
181:                if (payRoyalties) {
182:                    uint256 salePrice = outputAmount / sells[i].tokenIds.length;
183:                    for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
184:                        // get the royalty fee and recipient
185:                        (uint256 royaltyFee, address royaltyRecipient) =
186:                            getRoyalty(sells[i].nft, sells[i].tokenIds[j], salePrice);
187:
188:                        if (royaltyFee > 0) {
189:                            // transfer the royalty fee to the royalty recipient
190:                            royaltyRecipient.safeTransferETH(royaltyFee);
191:                        }
192:                    }
193:                }
194:            } else {
195:                // execute the sell against a private pool
196:                PrivatePool(sells[i].pool).sell(
197:                    sells[i].tokenIds, sells[i].tokenWeights, sells[i].proof, sells[i].stolenNftProofs
198:                );
199:            }
200:        }
```
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..12218d6 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -157,33 +157,35 @@ contract EthRouter is ERC721TokenReceiver {

         // loop through and execute the sells
         for (uint256 i = 0; i < sells.length; i++) {
+            Sell calldata _sell = sells[i];
+            uint256[] calldata _tokenIds = _sell.tokenIds;
             // transfer the NFTs into the router from the caller
-            for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
-                ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
+            for (uint256 j = 0; j < _tokenIds.length; j++) {
+                ERC721(_sell.nft).safeTransferFrom(msg.sender, address(this), _tokenIds[j]);
             }

             // approve the pair to transfer NFTs from the router
-            ERC721(sells[i].nft).setApprovalForAll(sells[i].pool, true);
+            ERC721(_sell.nft).setApprovalForAll(_sell.pool, true);

-            if (sells[i].isPublicPool) {
+            if (_sell.isPublicPool) {
                 // exceute the sell against a public pool
-                uint256 outputAmount = Pair(sells[i].pool).nftSell(
-                    sells[i].tokenIds,
+                uint256 outputAmount = Pair(_sell.pool).nftSell(
+                    _tokenIds,
                     0,
                     0,
-                    sells[i].publicPoolProofs,
+                    _sell.publicPoolProofs,
                     // ReservoirOracle.Message[] is the exact same as IStolenNftOracle.Message[] and can be
                     // decoded/encoded 1-to-1.
-                    abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))
+                    abi.decode(abi.encode(_sell.stolenNftProofs), (ReservoirOracle.Message[]))
                 );

                 // pay the royalties if seller has opted-in
                 if (payRoyalties) {
-                    uint256 salePrice = outputAmount / sells[i].tokenIds.length;
-                    for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
+                    uint256 salePrice = outputAmount / _tokenIds.length;
+                    for (uint256 j = 0; j < _tokenIds.length; j++) {
                         // get the royalty fee and recipient
                         (uint256 royaltyFee, address royaltyRecipient) =
-                            getRoyalty(sells[i].nft, sells[i].tokenIds[j], salePrice);
+                            getRoyalty(_sell.nft, _tokenIds[j], salePrice);

                         if (royaltyFee > 0) {
                             // transfer the royalty fee to the royalty recipient
@@ -193,8 +195,8 @@ contract EthRouter is ERC721TokenReceiver {
                 }
             } else {
                 // execute the sell against a private pool
-                PrivatePool(sells[i].pool).sell(
-                    sells[i].tokenIds, sells[i].tokenWeights, sells[i].proof, sells[i].stolenNftProofs
+                PrivatePool(_sell.pool).sell(
+                    _tokenIds, _sell.tokenWeights, _sell.proof, _sell.stolenNftProofs
                 );
             }
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L261-L287

### Cache calldata pointer for `changes[i]` and memory pointers for `changes[i].inputTokenIds` & `changes[i].outputTokenIds`

*Gas Savings for `EthRouter.change`, obtained via protocol's tests: Avg 4314 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  284857  |  298879  |  217295 |    4     |
| After  |  279105  |  293127  |  212981 |    4     |

```solidity
File: src/EthRouter.sol
261:        for (uint256 i = 0; i < changes.length; i++) {
262:            Change memory _change = changes[i];
263:
264:            // transfer NFTs from caller
265:            for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
266:                ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);
267:            }
268:
269:            // approve pair to transfer NFTs from router
270:            ERC721(_change.nft).setApprovalForAll(_change.pool, true);
271:
272:            // execute change
273:            PrivatePool(_change.pool).change{value: msg.value}(
274:                _change.inputTokenIds,
275:                _change.inputTokenWeights,
276:                _change.inputProof,
277:                _change.stolenNftProofs,
278:                _change.outputTokenIds,
279:                _change.outputTokenWeights,
280:                _change.outputProof
281:            );
282:
283:            // transfer NFTs to caller
284:            for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
285:                ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);
286:            }
287:        }
```
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..c63c2f8 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -259,11 +259,13 @@ contract EthRouter is ERC721TokenReceiver {

         // loop through and execute the changes
         for (uint256 i = 0; i < changes.length; i++) {
-            Change memory _change = changes[i];
+            Change calldata _change = changes[i];
+            uint256[] memory _inputTokenIds = _change.inputTokenIds;
+            uint256[] memory _outputTokenIds = _change.outputTokenIds;

             // transfer NFTs from caller
-            for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
-                ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);
+            for (uint256 j = 0; j < _inputTokenIds.length; j++) {
+                ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _inputTokenIds[j]);
             }

             // approve pair to transfer NFTs from router
@@ -271,18 +273,18 @@ contract EthRouter is ERC721TokenReceiver {

             // execute change
             PrivatePool(_change.pool).change{value: msg.value}(
-                _change.inputTokenIds,
+                _inputTokenIds,
                 _change.inputTokenWeights,
                 _change.inputProof,
                 _change.stolenNftProofs,
-                _change.outputTokenIds,
+                _outputTokenIds,
                 _change.outputTokenWeights,
                 _change.outputProof
             );

             // transfer NFTs to caller
-            for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
-                ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);
+            for (uint256 j = 0; j < _outputTokenIds.length; j++) {
+                ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _outputTokenIds[j]);
             }
         }
```

## Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as `memory`, that value will be copied into memory. When you specify the location as `calldata`, the value will stay static within calldata. If the value is a large, complex type, using `memory` may result in extra memory expansion costs.

**Note: We are able to change these instances from memory to calldata without causing a `stack too deep` error.**

Total Instances: `4`

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385-L393

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L661-L664

*Gas Savings for `PrivatePool.change`, obtained via protocol's tests: Avg 1489 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  116432  |  142008  |  82138  |    17    |
| After  |  114756  |  139483  |  80649  |    17    |

*Gas Savings for `PrivatePool.execute`, obtained via protocol's tests: Avg 340 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  18340   |  36348   |  18890  |    6     |
| After  |  17976   |  35939   |  18550  |    6     |

**Note: `PrivatePool.sumWeightsAndValidateProof` is called by the functions below**.

*Gas Savings for `PrivatePool.buy`, obtained via protocol's tests: Avg 147 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  74037   |  158864  |  70884  |    24    |
| After  |  73887   |  158708  |  70737  |    24    |

*Gas Savings for `PrivatePool.sell`, obtained via protocol's tests: Avg 171 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  50139   |  170448  |  81969  |    25    |
| After  |  49989   |  170256  |  81798  |    25    |

```solidity
File: src/PrivatePool.sol
385:    function change(
386:        uint256[] memory inputTokenIds,
387:        uint256[] memory inputTokenWeights,
388:        MerkleMultiProof memory inputProof,
389:        IStolenNftOracle.Message[] memory stolenNftProofs,
390:        uint256[] memory outputTokenIds,

459:    function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

661:    function sumWeightsAndValidateProof(
662:        uint256[] memory tokenIds,
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..914dace 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -383,11 +383,11 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @param outputTokenWeights The weights of the NFTs to receive.
     /// @param outputProof The merkle proof for the weights of each NFT to receive.
     function change(
-        uint256[] memory inputTokenIds,
+        uint256[] calldata inputTokenIds,
         uint256[] memory inputTokenWeights,
         MerkleMultiProof memory inputProof,
         IStolenNftOracle.Message[] memory stolenNftProofs,
-        uint256[] memory outputTokenIds,
+        uint256[] calldata outputTokenIds,
         uint256[] memory outputTokenWeights,
         MerkleMultiProof memory outputProof
     ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {
@@ -456,7 +456,7 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @param target The address of the target contract.
     /// @param data The data to send to the target contract.
     /// @return returnData The return data of the transaction.
-    function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
+    function execute(address target, bytes calldata data) public payable onlyOwner returns (bytes memory) {
         // call the target with the value and data
         (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

@@ -659,7 +659,7 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @param proof The merkle proof for the weights of each NFT.
     /// @return sum The sum of the weights of each NFT.
     function sumWeightsAndValidateProof(
-        uint256[] memory tokenIds,
+        uint256[] calldata tokenIds,
         uint256[] memory tokenWeights,
         MerkleMultiProof memory proof
     ) public view returns (uint256) {
```

## State variables can be cached instead of re-reading them from storage
Caching of a state variable replaces each `Gwarmaccess (100 gas)` with a much cheaper stack read.

**Note: Some view functions are included below since they are called within state mutating functions**

Total Instances: `16`

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L742-L746

### Cache `baseToken` to save 1 SLOAD
```solidity
File: src/PrivatePool.sol
742:    function price() public view returns (uint256) {
743:        // ensure that the exponent is always to 18 decimals of accuracy
744:        uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals()); // @audit: 1st & 2nd sload
745:        return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;
746:    }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..2f747fc 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -741,7 +741,8 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @return price The price of the pool.
     function price() public view returns (uint256) {
         // ensure that the exponent is always to 18 decimals of accuracy
-        uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
+        address _baseToken = baseToken;
+        uint256 exponent = _baseToken == address(0) ? 18 : (36 - ERC20(_baseToken).decimals());
         return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;
     }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L731-L738

### Cache `baseToken` to save 1 SLOAD
```solidity
File: src/PrivatePool.sol
    function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) { 
        // multiply the changeFee to get the fee per NFT (4 decimals of accuracy)
        uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4; // @audit: 1st and 2nd sload
        uint256 feePerNft = changeFee * 10 ** exponent;

        feeAmount = inputAmount * feePerNft / 1e18;
        protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;
    }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..aba824f 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -730,7 +730,8 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @return protocolFeeAmount The protocol fee amount.
     function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
         // multiply the changeFee to get the fee per NFT (4 decimals of accuracy)
-        uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;
+        address _baseToken = baseToken;
+        uint256 exponent = _baseToken == address(0) ? 18 - 4 : ERC20(_baseToken).decimals() - 4;
         uint256 feePerNft = changeFee * 10 ** exponent;

         feeAmount = inputAmount * feePerNft / 1e18;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L661-L684

### Cache `merkleRoot` to save 1 SLOAD
```solidity
File: src/PrivatePool.sol
661:    function sumWeightsAndValidateProof(
662:        uint256[] memory tokenIds,
663:        uint256[] memory tokenWeights,
664:        MerkleMultiProof memory proof
665:    ) public view returns (uint256) {
666:        // if the merkle root is not set then set the weight of each nft to be 1e18
667:        if (merkleRoot == bytes32(0)) { // @audit: 1st sload
668:            return tokenIds.length * 1e18;
669:        }
670:
671:        uint256 sum;
672:        bytes32[] memory leafs = new bytes32[](tokenIds.length);
673:        for (uint256 i = 0; i < tokenIds.length; i++) {
674:            // create the leaf for the merkle proof
675:            leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
676:
677:            // sum each token weight
678:            sum += tokenWeights[i];
679:        }
680:
681:        // validate that the weights are valid against the merkle proof
682:        if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)) { // @audit: 2nd sload
683:            revert InvalidMerkleProof();
684:        }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..7385ea9 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -664,7 +664,8 @@ contract PrivatePool is ERC721TokenReceiver {
         MerkleMultiProof memory proof
     ) public view returns (uint256) {
         // if the merkle root is not set then set the weight of each nft to be 1e18
-        if (merkleRoot == bytes32(0)) {
+        bytes32 _merkleRoot = merkleRoot;
+        if (_merkleRoot == bytes32(0)) {
             return tokenIds.length * 1e18;
         }

@@ -679,7 +680,7 @@ contract PrivatePool is ERC721TokenReceiver {
         }

         // validate that the weights are valid against the merkle proof
-        if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)) {
+        if (!MerkleProofLib.verifyMultiProof(proof.proof, _merkleRoot, leafs, proof.flags)) {
             revert InvalidMerkleProof();
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635-L651

### Cache `baseToken` to save 2 SLOADs
```solidity
File: src/PrivatePool.sol
635:        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount(); // @audit: 1st sload
636:
637:        // transfer the NFT to the borrower
638:        ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);
639:
640:        // call the borrower
641:        bool success =
642:            receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");
643:
644:        // check that flashloan was successful
645:        if (!success) revert FlashLoanFailed();
646:
647:        // transfer the NFT from the borrower
648:        ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);
649:
650:        // transfer the fee from the borrower
651:        if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee); // @audit: 2nd & 3rd sload
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..c8b3cc1 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -632,7 +632,8 @@ contract PrivatePool is ERC721TokenReceiver {
         uint256 fee = flashFee(token, tokenId);

         // if base token is ETH then check that caller sent enough for the fee
-        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
+        address _baseToken = baseToken;
+        if (_baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

         // transfer the NFT to the borrower
         ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);
@@ -648,7 +649,7 @@ contract PrivatePool is ERC721TokenReceiver {
         ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);

         // transfer the fee from the borrower
-        if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);
+        if (_baseToken != address(0)) ERC20(_baseToken).transferFrom(msg.sender, address(this), fee);

         return success;
     }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489-L503

### Cache `baseToken` to save 3 SLOADs
```solidity
File: src/PrivatePool.sol
489:        if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) { // @audit: 1st & 2nd sload
490:            revert InvalidEthAmount();
491:        }
492:
493:        // ~~~ Interactions ~~~ //
494:
495:        // transfer the nfts from the caller
496:        for (uint256 i = 0; i < tokenIds.length; i++) {
497:            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
498:        }
499:
500:        if (baseToken != address(0)) { // @audit: 3rd sload
501:            // transfer the base tokens from the caller
502:            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount); // @audit: 4th sload
503:        }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..92349e1 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -486,7 +486,8 @@ contract PrivatePool is ERC721TokenReceiver {

         // ensure the caller sent a valid amount of ETH if base token is ETH or that the caller sent 0 ETH if base token
         // is not ETH
-        if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
+        address _baseToken = baseToken;
+        if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && _baseToken != address(0))) {
             revert InvalidEthAmount();
         }

@@ -497,9 +498,9 @@ contract PrivatePool is ERC721TokenReceiver {
             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
         }

-        if (baseToken != address(0)) {
+        if (_baseToken != address(0)) {
             // transfer the base tokens from the caller
-            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);
+            ERC20(_baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397-L426

### Cache `baseToken` to save 3 SLOADs
```solidity
File: src/PrivatePool.sol
397:        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount(); // @audit: 1st sload
...
421:        if (baseToken != address(0)) { // @audit: 2nd sload
422:            // transfer the fee amount of base tokens from the caller
423:            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount); // @audit: 3rd sload
424:
425:            // if the protocol fee is non-zero then transfer the protocol fee to the factory
426:            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount); // @audit: 4th sload
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..912790f 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -394,7 +394,8 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Checks ~~~ //

         // check that the caller sent 0 ETH if base token is not ETH
-        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
+        address _baseToken = baseToken;
+        if (_baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

         // check that NFTs are not stolen
         if (useStolenNftOracle) {
@@ -418,12 +419,12 @@ contract PrivatePool is ERC721TokenReceiver {

         // ~~~ Interactions ~~~ //

-        if (baseToken != address(0)) {
+        if (_baseToken != address(0)) {
             // transfer the fee amount of base tokens from the caller
-            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);
+            ERC20(_baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);

             // if the protocol fee is non-zero then transfer the protocol fee to the factory
-            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);
+            if (protocolFeeAmount > 0) ERC20(_baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);
         } else {
             // check that the caller sent enough ETH to cover the fee amount and protocol fee
             if (msg.value < feeAmount + protocolFeeAmount) revert InvalidEthAmount();
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L357-L369

### Cache `baseToken` to save 2 SLOADs
```solidity
File: src/PrivatePool.sol
357:        if (baseToken == address(0)) { // @audit: 1st sload
358:            // transfer ETH to the caller
359:            msg.sender.safeTransferETH(netOutputAmount);
360:
361:            // if the protocol fee is set then pay the protocol fee
362:            if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
363:        } else {
364:            // transfer base tokens to the caller
365:            ERC20(baseToken).transfer(msg.sender, netOutputAmount); // @audit: 2nd sload
366:
367:            // if the protocol fee is set then pay the protocol fee
368:            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount); // @audit: 3rd sload
369:        }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..2511385 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -353,8 +353,9 @@ contract PrivatePool is ERC721TokenReceiver {

         // subtract the royalty fee amount from the net output amount
         netOutputAmount -= royaltyFeeAmount;
-
-        if (baseToken == address(0)) {
+
+        address _baseToken = baseToken;
+        if (_baseToken == address(0)) {
             // transfer ETH to the caller
             msg.sender.safeTransferETH(netOutputAmount);

@@ -362,10 +363,10 @@ contract PrivatePool is ERC721TokenReceiver {
             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
         } else {
             // transfer base tokens to the caller
-            ERC20(baseToken).transfer(msg.sender, netOutputAmount);
+            ERC20(_baseToken).transfer(msg.sender, netOutputAmount);

             // if the protocol fee is set then pay the protocol fee
-            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
+            if (protocolFeeAmount > 0) ERC20(_baseToken).safeTransfer(address(factory), protocolFeeAmount);
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225-L259

### Cache `baseToken` to save 3 SLOADs.
```solidity
File: src/PrivatePool.sol
225:        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount(); // @audit: 1st sload
...        
254:        if (baseToken != address(0)) { // @audit: 2nd sload
255:            // transfer the base token from the caller to the contract
256:            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount); // @audit: 3rd sload
257:
258:            // if the protocol fee is set then pay the protocol fee
259:            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount); // @audit: 4th sload
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..5cc3318 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -216,19 +216,22 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Checks ~~~ //

         // calculate the sum of weights of the NFTs to buy
-        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
+        address _baseToken = baseToken;
+        {
+            uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

-        // calculate the required net input amount and fee amount
-        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);
+            // calculate the required net input amount and fee amount
+            (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

-        // check that the caller sent 0 ETH if the base token is not ETH
-        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
+            // check that the caller sent 0 ETH if the base token is not ETH
+            if (_baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

-        // ~~~ Effects ~~~ //
+            // ~~~ Effects ~~~ //

-        // update the virtual reserves
-        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
-        virtualNftReserves -= uint128(weightSum);
+            // update the virtual reserves
+            virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
+            virtualNftReserves -= uint128(weightSum);
+        }

         // ~~~ Interactions ~~~ //

@@ -251,12 +254,12 @@ contract PrivatePool is ERC721TokenReceiver {
         // add the royalty fee amount to the net input aount
         netInputAmount += royaltyFeeAmount;

-        if (baseToken != address(0)) {
+        if (_baseToken != address(0)) {
             // transfer the base token from the caller to the contract
-            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);
+            ERC20(_baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);

             // if the protocol fee is set then pay the protocol fee
-            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
+            if (protocolFeeAmount > 0) ERC20(_baseToken).safeTransfer(address(factory), protocolFeeAmount);
         } else {
             // check that the caller sent enough ETH to cover the net required input
             if (msg.value < netInputAmount) revert InvalidEthAmount();
```


## Cache state variables outside of loop to avoid reading storage on every iteration
Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a `Gwarmaccess (100 gas)` per loop iteration.

**Note: Due to `stack too deep` errors, we will not be able to cache all the state variables read within the loops.** 

Total Instances: `6`

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238-L271

### Cache `payRoyalties` to avoid storage reads on each loop iteration
```solidity
File: src/PrivatePool.sol
238:        for (uint256 i = 0; i < tokenIds.length; i++) {
239:            // transfer the NFT to the caller
240:            ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
241:
242:            if (payRoyalties) { // @audit 1st sload + on every iteration
...
271:        if (payRoyalties) { // @audit: 2nd sload
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..5d46070 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -216,30 +216,33 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Checks ~~~ //

         // calculate the sum of weights of the NFTs to buy
-        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
+        {
+            uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

-        // calculate the required net input amount and fee amount
-        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);
+            // calculate the required net input amount and fee amount
+            (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

-        // check that the caller sent 0 ETH if the base token is not ETH
-        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
+            // check that the caller sent 0 ETH if the base token is not ETH
+            if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

-        // ~~~ Effects ~~~ //
+            // ~~~ Effects ~~~ //

-        // update the virtual reserves
-        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
-        virtualNftReserves -= uint128(weightSum);
+            // update the virtual reserves
+            virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
+            virtualNftReserves -= uint128(weightSum);
+        }

         // ~~~ Interactions ~~~ //

         // calculate the sale price (assume it's the same for each NFT even if weights differ)
         uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
         uint256 royaltyFeeAmount = 0;
+        bool _payRoyalties = payRoyalties;
         for (uint256 i = 0; i < tokenIds.length; i++) {
             // transfer the NFT to the caller
             ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

-            if (payRoyalties) {
+            if (_payRoyalties) {
                 // get the royalty fee for the NFT
                 (uint256 royaltyFee,) = _getRoyalty(tokenIds[i], salePrice);

@@ -268,7 +271,7 @@ contract PrivatePool is ERC721TokenReceiver {
             if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);
         }

-        if (payRoyalties) {
+        if (_payRoyalties) {
             for (uint256 i = 0; i < tokenIds.length; i++) {
                 // get the royalty fee for the NFT
                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238-L240

### Cache `nft` to avoid storage reads on each loop iteration
```solidity
File: src/PrivatePool.sol
238:        for (uint256 i = 0; i < tokenIds.length; i++) {
239:            // transfer the NFT to the caller
240:            ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]); // @audit: sload on every iteration
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..257beba 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -216,28 +216,31 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Checks ~~~ //

         // calculate the sum of weights of the NFTs to buy
-        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
+        {
+            uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

-        // calculate the required net input amount and fee amount
-        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);
+            // calculate the required net input amount and fee amount
+            (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

-        // check that the caller sent 0 ETH if the base token is not ETH
-        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
+            // check that the caller sent 0 ETH if the base token is not ETH
+            if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

-        // ~~~ Effects ~~~ //
+            // ~~~ Effects ~~~ //

-        // update the virtual reserves
-        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
-        virtualNftReserves -= uint128(weightSum);
+            // update the virtual reserves
+            virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
+            virtualNftReserves -= uint128(weightSum);
+        }

         // ~~~ Interactions ~~~ //

         // calculate the sale price (assume it's the same for each NFT even if weights differ)
         uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
         uint256 royaltyFeeAmount = 0;
+        address _nft = nft;
         for (uint256 i = 0; i < tokenIds.length; i++) {
             // transfer the NFT to the caller
-            ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
+            ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329-L333

### Cache `payRoyalties` to avoid storage reads on each loop iteration
```solidity
File: src/PrivatePool.sol
329:        for (uint256 i = 0; i < tokenIds.length; i++) {
330:            // transfer each nft from the caller
331:            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
332:
333:            if (payRoyalties) { // @audit: sload on every iteration
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..a41a1f5 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -307,30 +307,33 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Checks ~~~ //

         // calculate the sum of weights of the NFTs to sell
-        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
+        {
+            uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

-        // calculate the net output amount and fee amount
-        (netOutputAmount, feeAmount, protocolFeeAmount) = sellQuote(weightSum);
+            // calculate the net output amount and fee amount
+            (netOutputAmount, feeAmount, protocolFeeAmount) = sellQuote(weightSum);

-        //  check the nfts are not stolen
-        if (useStolenNftOracle) {
-            IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
-        }
+            //  check the nfts are not stolen
+            if (useStolenNftOracle) {
+                IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
+            }

-        // ~~~ Effects ~~~ //
+            // ~~~ Effects ~~~ //

-        // update the virtual reserves
-        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
-        virtualNftReserves += uint128(weightSum);
+            // update the virtual reserves
+            virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
+            virtualNftReserves += uint128(weightSum);
+        }

         // ~~~ Interactions ~~~ //

         uint256 royaltyFeeAmount = 0;
+        bool _payRoyalties = payRoyalties;
         for (uint256 i = 0; i < tokenIds.length; i++) {
             // transfer each nft from the caller
             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

-            if (payRoyalties) {
+            if (_payRoyalties) {
                 // calculate the sale price (assume it's the same for each NFT even if weights differ)
                 uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329-L331

### Cache `nft` to avoid storage reads on each loop iteration
```solidity
File: src/PrivatePool.sol
329:        for (uint256 i = 0; i < tokenIds.length; i++) {
330:            // transfer each nft from the caller
331:            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]); // @audit: sload on every iteration
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..70009e1 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -307,28 +307,31 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Checks ~~~ //

         // calculate the sum of weights of the NFTs to sell
-        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
+        {
+            uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

-        // calculate the net output amount and fee amount
-        (netOutputAmount, feeAmount, protocolFeeAmount) = sellQuote(weightSum);
+            // calculate the net output amount and fee amount
+            (netOutputAmount, feeAmount, protocolFeeAmount) = sellQuote(weightSum);

-        //  check the nfts are not stolen
-        if (useStolenNftOracle) {
-            IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
-        }
+            //  check the nfts are not stolen
+            if (useStolenNftOracle) {
+                IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
+            }

-        // ~~~ Effects ~~~ //
+            // ~~~ Effects ~~~ //

-        // update the virtual reserves
-        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
-        virtualNftReserves += uint128(weightSum);
+            // update the virtual reserves
+            virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
+            virtualNftReserves += uint128(weightSum);
+        }

         // ~~~ Interactions ~~~ //

         uint256 royaltyFeeAmount = 0;
+        address _nft = nft;
         for (uint256 i = 0; i < tokenIds.length; i++) {
             // transfer each nft from the caller
-            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
+            ERC721(_nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441-L448

### Cache `nft` to avoid storage reads on each loop iteration
```solidity
File: src/PrivatePool.sol
441:        for (uint256 i = 0; i < inputTokenIds.length; i++) {
442:            ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]); // @audit: sload on every iteration
443:        }
444:
445:        // transfer the output nfts to the caller
446:        for (uint256 i = 0; i < outputTokenIds.length; i++) {
447:            ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]); // @audit: sload on every iteration
448:        }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..5e4e292 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -438,13 +438,14 @@ contract PrivatePool is ERC721TokenReceiver {
         }

         // transfer the input nfts from the caller
+        address _nft = nft;
         for (uint256 i = 0; i < inputTokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
+            ERC721(_nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
         }

         // transfer the output nfts to the caller
         for (uint256 i = 0; i < outputTokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
+            ERC721(_nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L496-L498

### Cache `nft` to avoid storage reads on each loop iteration
```solidity
File: src/PrivatePool.sol
496:        for (uint256 i = 0; i < tokenIds.length; i++) {
497:            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]); // @audit: sload on every iteration
498:        }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..65eee4c 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -493,8 +493,9 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Interactions ~~~ //

         // transfer the nfts from the caller
+        address _nft = nft;
         for (uint256 i = 0; i < tokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
+            ERC721(_nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
         }

         if (baseToken != address(0)) {
```

## Rearrange code to fail early
In the instance below, two gas-intensive internal functions are invoked before the `if` statement. If the check causes a revert, the gas consumed in the first two internal functions will not be refunded. Move the `if` statement to the top of the function to save gas for users that trigger a revert.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L219-L225

*Gas Savings for `PrivatePool.buy`, obtained via protocol's tests: Avg 327 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  74037   |  158864  |  70884  |    24    |
| After  |  74040   |  158867  |  70557  |    24    |

```solidity
File: src/PrivatePool.sol
219:        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
220:
221:        // calculate the required net input amount and fee amount
222:        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);
223:
224:        // check that the caller sent 0 ETH if the base token is not ETH
225:        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..7465103 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -215,15 +215,15 @@ contract PrivatePool is ERC721TokenReceiver {
     {
         // ~~~ Checks ~~~ //

+        // check that the caller sent 0 ETH if the base token is not ETH
+        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
+
         // calculate the sum of weights of the NFTs to buy
         uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

         // calculate the required net input amount and fee amount
         (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

-        // check that the caller sent 0 ETH if the base token is not ETH
-        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
-
         // ~~~ Effects ~~~ //
```

## `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables

Total Instances: `2`

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230-L231

*Gas Savings for `PrivatePool.buy`, obtained via protocol's tests: Avg 289 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  74037   |  158864  |  70884  |    24    |
| After  |  73713   |  158540  |  70595  |    24    |

```solidity
File: src/PrivatePool.sol
230:        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
231:        virtualNftReserves -= uint128(weightSum);
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..a0813a6 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -227,8 +227,8 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Effects ~~~ //

         // update the virtual reserves
-        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
-        virtualNftReserves -= uint128(weightSum);
+        virtualBaseTokenReserves = virtualBaseTokenReserves + uint128(netInputAmount - feeAmount - protocolFeeAmount);
+        virtualNftReserves = virtualNftReserves - uint128(weightSum);

         // ~~~ Interactions ~~~ //
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323-L324

*Gas Savings for `PrivatePool.sell`, obtained via protocol's tests: Avg 255 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  50139   |  170448  |  81969  |    25    |
| After  |  49891   |  170138  |  81714  |    25    |

```solidity
File: src/PrivatePool.sol
323:        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
324:        virtualNftReserves += uint128(weightSum);
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..8a99e47 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -320,8 +320,8 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Effects ~~~ //

         // update the virtual reserves
-        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
-        virtualNftReserves += uint128(weightSum);
+        virtualBaseTokenReserves = virtualBaseTokenReserves - uint128(netOutputAmount + protocolFeeAmount + feeAmount);
+        virtualNftReserves = virtualNftReserves + uint128(weightSum);

         // ~~~ Interactions ~~~ //
```

## `If` statements that use `&&` can be refactored into nested `if` statements

Total Instances: `9`

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277

*Gas Savings for `PrivatePool.buy`, obtained via protocol's tests: Avg 25 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  74037   |  158864  |  70884  |    24    |
| After  |  74011   |  158793  |  70859  |    24    |

```solidity
File: src/PrivatePool.sol
225:        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

277:        if (royaltyFee > 0 && recipient != address(0)) {
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..73492e8 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -222,7 +222,11 @@ contract PrivatePool is ERC721TokenReceiver {
         (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

         // check that the caller sent 0 ETH if the base token is not ETH
-        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
+        if (baseToken != address(0)) {
+            if (msg.value > 0) {
+                revert InvalidEthAmount();
+            }
+        }

         // ~~~ Effects ~~~ //

@@ -274,11 +278,13 @@ contract PrivatePool is ERC721TokenReceiver {
                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

                 // transfer the royalty fee to the recipient if it's greater than 0
-                if (royaltyFee > 0 && recipient != address(0)) {
-                    if (baseToken != address(0)) {
-                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);
-                    } else {
-                        recipient.safeTransferETH(royaltyFee);
+                if (royaltyFee > 0) {
+                    if (recipient != address(0)) {
+                        if (baseToken != address(0)) {
+                            ERC20(baseToken).safeTransfer(recipient, royaltyFee);
+                        } else {
+                            recipient.safeTransferETH(royaltyFee);
+                        }
                     }
                 }
             }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344-L350

*Gas Savings for `PrivatePool.sell`, obtained via protocol's tests: Avg 29 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  50139   |  170448  |  81969  |    25    |
| After  |  50139   |  170394  |  81940  |    25    |

```solidity
File: src/PrivatePool.sol
344:                if (royaltyFee > 0 && recipient != address(0)) {
345:                    if (baseToken != address(0)) {
346:                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);
347:                    } else {
348:                        recipient.safeTransferETH(royaltyFee);
349:                    }
350:                }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..a3218eb 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -341,11 +341,13 @@ contract PrivatePool is ERC721TokenReceiver {
                 royaltyFeeAmount += royaltyFee;

                 // transfer the royalty fee to the recipient if it's greater than 0
-                if (royaltyFee > 0 && recipient != address(0)) {
-                    if (baseToken != address(0)) {
-                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);
-                    } else {
-                        recipient.safeTransferETH(royaltyFee);
+                if (royaltyFee > 0) {
+                    if (recipient != address(0)) {
+                        if (baseToken != address(0)) {
+                            ERC20(baseToken).safeTransfer(recipient, royaltyFee);
+                        } else {
+                            recipient.safeTransferETH(royaltyFee);
+                        }
                     }
                 }
             }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397

*Gas Savings for `PrivatePool.change`, obtained via protocol's tests: Avg 25 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  116432  |  142008  |  82138  |    17    |
| After  |  116406  |  141982  |  82113  |    17    |

```solidity
File: src/PrivatePool.sol
397:        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..e9bd8f2 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -394,7 +394,11 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Checks ~~~ //

         // check that the caller sent 0 ETH if base token is not ETH
-        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
+        if (baseToken != address(0)) {
+            if (msg.value > 0) {
+                revert InvalidEthAmount();
+            }
+        }

         // check that NFTs are not stolen
         if (useStolenNftOracle) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635

*Gas Savings for `PrivatePool.flashLoan`, obtained via protocol's tests: Avg 16 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  83063   |  103206  |  83063  |    2     |
| After  |  83047   |  103185  |  83047  |    2     |

```solidity
File: src/PrivatePool.sol
635:        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..fe5c440 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -632,7 +632,11 @@ contract PrivatePool is ERC721TokenReceiver {
         uint256 fee = flashFee(token, tokenId);

         // if base token is ETH then check that caller sent enough for the fee
-        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
+        if (baseToken == address(0)) {
+            if (msg.value < fee) {
+                revert InvalidEthAmount();
+            }
+        }

         // transfer the NFT to the borrower
         ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101-L103

*Gas Savings for `EthRouter.buy`, obtained via protocol's tests: Avg 10 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  187054  |  397581  |  199750 |    7     |
| After  |  187044  |  397569  |  199740 |    7     |

```solidity
File: src/EthRouter.sol
101:        if (block.timestamp > deadline && deadline != 0) {
102:            revert DeadlinePassed();
103:        }
```
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..41ef7bf 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -98,8 +98,10 @@ contract EthRouter is ERC721TokenReceiver {
     /// @param payRoyalties Whether to pay royalties or not.
     function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
         // check that the deadline has not passed (if any)
-        if (block.timestamp > deadline && deadline != 0) {
-            revert DeadlinePassed();
+        if (block.timestamp > deadline) {
+            if (deadline != 0) {
+                revert DeadlinePassed();
+            }
         }
```

The instances below are similar to the one above:

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154-L156

*Gas Savings for `EthRouter.sell`, obtained via protocol's tests: Avg 11 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  217300  |  402940  |  232102 |    7     |
| After  |  217290  |  402928  |  232091 |    7     |

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228-L230

*Gas Savings for `EthRouter.deposit`, obtained via protocol's tests: Avg 12 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  2371    |  114072  |  29900  |    4     |
| After  |  2359    |  114060  |  29888  |    4     |

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256-L258

*Gas Savings for `EthRouter.change`, obtained via protocol's tests: Avg 12 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  284857  |  298879  |  217295 |    4     |
| After  |  284845  |  298867  |  217283 |    4     |

## Use assembly for loops
In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are `scratch space (0x00-0x20)`, the `free memory pointer (0x40)`, and the `zero slot (0x60)`. This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

**Note that in order to do this optimization safely we will need to cache and restore the `free memory pointer` after the loop. We will also set the `zero slot (0x60)` back to 0.**

*The [final diffs](https://gist.github.com/0xJCN/b03b5f1f8cabc937c8bbe4d4a46b8d47) have comments explaining the assembly code*.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L119-L121

*Gas Savings for `Factory.create`, obtained via protocol's tests: Avg 129 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  161124  |  245619  |  148801 |    35    |
| After  |  161112  |  243941  |  148672 |    35    |

```solidity
File: src/Factory.sol
119:        for (uint256 i = 0; i < tokenIds.length; i++) {
120:            ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
121:        }
```
```diff
diff --git a/src/Factory.sol b/src/Factory.sol
index 09cbb4e..c2e06f6 100644
--- a/src/Factory.sol
+++ b/src/Factory.sol
@@ -116,8 +116,26 @@ contract Factory is ERC721, Owned {
         }

         // deposit the nfts from the caller into the pool
-        for (uint256 i = 0; i < tokenIds.length; i++) {
-            ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
+        assembly {
+            if mload(tokenIds) {
+                let memptr := mload(0x40)
+                let end := add(add(tokenIds, 0x20), mul(0x20, mload(tokenIds)))
+                let i := add(tokenIds, 0x20)
+                mstore(0x00, 0x42842e0e)
+                mstore(0x20, caller())
+                mstore(0x40, privatePool)
+                for {} 1 {} {
+                    mstore(0x60, mload(i))
+                    let success := call(gas(), calldataload(0x24), 0x00, 0x1c, 0x64, 0x00, 0x00)
+                    if iszero(success) {
+                        revert(0, 0)
+                    }
+                    i := add(i, 0x20)
+                    if iszero(lt(i, end)) { break }
+                }
+                mstore(0x40, memptr)
+                mstore(0x60, 0x00)
+            }
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L134-L137

*Gas Savings for `EthRouter.buy`, obtained via protocol's tests: Avg 5728 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  187054  |  397581  |  199750 |    7     |
| After  |  182465  |  384540  |  194022 |    7     |

```solidity
File: src/EthRouter.sol
134:            for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
135:                // transfer the NFT to the caller
136:                ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
137:            }
```
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..4ea89f5 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -131,9 +131,28 @@ contract EthRouter is ERC721TokenReceiver {
                 );
             }

-            for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
-                // transfer the NFT to the caller
-                ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
+            Buy calldata _buy = buys[i];
+            uint256[] calldata _tokenIds = _buy.tokenIds;
+            assembly {
+                if _tokenIds.length {
+                    let memptr := mload(0x40)
+                    let end := add(_tokenIds.offset, mul(0x20, _tokenIds.length))
+                    let j := _tokenIds.offset
+                    mstore(0x00, 0x42842e0e)
+                    mstore(0x20, address())
+                    mstore(0x40, caller())
+                    for {} 1 {} {
+                        mstore(0x60, calldataload(j))
+                        let success := call(gas(), calldataload(add(_buy, 0x20)), 0x00, 0x1c, 0x64, 0x00, 0x00)
+                        if iszero(success) {
+                            revert(0, 0)
+                        }
+                        j := add(j, 0x20)
+                        if iszero(lt(j, end)) { break }
+                    }
+                    mstore(0x40, memptr)
+                    mstore(0x60, 0x00)
+                }
             }
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L161-L163

*Gas Savings for `EthRouter.sell`, obtained via protocol's tests: Avg 4735 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  217300  |  402940  |  232102 |    7     |
| After  |  212706  |  395062  |  227367 |    7     |

```solidity
File: src/EthRouter.sol
161:            for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
162:                ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
163:            }
```
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..c1a07ab 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -158,8 +158,30 @@ contract EthRouter is ERC721TokenReceiver {
         // loop through and execute the sells
         for (uint256 i = 0; i < sells.length; i++) {
             // transfer the NFTs into the router from the caller
-            for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
-                ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
+            {
+                Sell calldata _sell = sells[i];
+                uint256[] calldata _tokenIds = _sell.tokenIds;
+                assembly {
+                    if _tokenIds.length {
+                        let memptr := mload(0x40)
+                        let end := add(_tokenIds.offset, mul(0x20, _tokenIds.length))
+                        let j := _tokenIds.offset
+                        mstore(0x00, 0x42842e0e)
+                        mstore(0x20, caller())
+                        mstore(0x40, address())
+                        for {} 1 {} {
+                            mstore(0x60, calldataload(j))
+                            let success := call(gas(), calldataload(add(_sell, 0x20)), 0x00, 0x1c, 0x64, 0x00, 0x00)
+                            if iszero(success) {
+                                revert(0, 0)
+                            }
+                            j := add(j, 0x20)
+                            if iszero(lt(j, end)) { break }
+                        }
+                        mstore(0x40, memptr)
+                        mstore(0x60, 0x00)
+                    }
+                }
             }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L239-L241

*Gas Savings for `EthRouter.deposit`, obtained via protocol's tests: Avg 211 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  2371    |  114072  |  29900  |    4     |
| After  |  2371    |  113228  |  29689  |    4     |

```solidity
File: src/EthRouter.sol
239:        for (uint256 i = 0; i < tokenIds.length; i++) {
240:            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
241:        }
```
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..3006dcb 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -236,8 +236,26 @@ contract EthRouter is ERC721TokenReceiver {
         }

         // transfer NFTs from caller
-        for (uint256 i = 0; i < tokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
+        assembly {
+            if tokenIds.length {
+                let memptr := mload(0x40)
+                let end := add(tokenIds.offset, mul(0x20, tokenIds.length))
+                let i := tokenIds.offset
+                mstore(0x00, 0x42842e0e)
+                mstore(0x20, caller())
+                mstore(0x40, address())
+                for {} 1 {} {
+                    mstore(0x60, calldataload(i))
+                    let success := call(gas(), calldataload(0x24), 0x00, 0x1c, 0x64, 0x00, 0x00)
+                    if iszero(success) {
+                        revert(0, 0)
+                    }
+                    i := add(i, 0x20)
+                    if iszero(lt(i, end)) { break }
+                }
+                mstore(0x40, memptr)
+                mstore(0x60, 0x00)
+            }
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L265-L286

*Gas Savings for `EthRouter.change`, obtained via protocol's tests: Avg 6587 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  284857  |  298879  |  217295 |    4     |
| After  |  276074  |  290096  |  210708 |    4     |

```solidity
File: src/EthRouter.sol
265:            for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
266:                ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);
267:            }
...            
284:            for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
285:                ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);
286:            }
```
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..250b83b 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -260,10 +260,30 @@ contract EthRouter is ERC721TokenReceiver {
         // loop through and execute the changes
         for (uint256 i = 0; i < changes.length; i++) {
             Change memory _change = changes[i];
+            uint256[] memory _inputTokenIds = _change.inputTokenIds;
+            uint256[] memory _outputTokenIds = _change.outputTokenIds;

             // transfer NFTs from caller
-            for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
-                ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);
+            assembly {
+                if mload(_inputTokenIds) {
+                    let memptr := mload(0x40)
+                    let end := add(add(_inputTokenIds, 0x20), mul(0x20, mload(_inputTokenIds)))
+                    let j := add(_inputTokenIds, 0x20)
+                    mstore(0x00, 0x42842e0e)
+                    mstore(0x20, caller())
+                    mstore(0x40, address())
+                    for {} 1 {} {
+                        mstore(0x60, mload(j))
+                        let success := call(gas(), mload(add(_change, 0x20)), 0x00, 0x1c, 0x64, 0x00, 0x00)
+                        if iszero(success) {
+                            revert(0, 0)
+                        }
+                        j := add(j, 0x20)
+                        if iszero(lt(j, end)) { break }
+                    }
+                    mstore(0x40, memptr)
+                    mstore(0x60, 0x00)
+                }
             }

             // approve pair to transfer NFTs from router
@@ -281,8 +301,26 @@ contract EthRouter is ERC721TokenReceiver {
             );

             // transfer NFTs to caller
-            for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
-                ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);
+            assembly {
+                if mload(_outputTokenIds) {
+                    let memptr := mload(0x40)
+                    let end := add(add(_outputTokenIds, 0x20), mul(0x20, mload(_outputTokenIds)))
+                    let j := add(_outputTokenIds, 0x20)
+                    mstore(0x00, 0x42842e0e)
+                    mstore(0x20, address())
+                    mstore(0x40, caller())
+                    for {} 1 {} {
+                        mstore(0x60, mload(j))
+                        let success := call(gas(), mload(add(_change, 0x20)), 0x00, 0x1c, 0x64, 0x00, 0x00)
+                        if iszero(success) {
+                            revert(0, 0)
+                        }
+                        j := add(j, 0x20)
+                        if iszero(lt(j, end)) { break }
+                    }
+                    mstore(0x40, memptr)
+                    mstore(0x60, 0x00)
+                }
             }
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441-L448

*Gas Savings for `PrivatePool.change`, obtained via protocol's tests: Avg 2388 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  116432  |  142008  |  82138  |    17    |
| After  |  113218  |  136466  |  79750  |    17    |

```solidity
File: src/PrivatePool.sol
441:        for (uint256 i = 0; i < inputTokenIds.length; i++) {
442:            ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
443:        }
444:
445:        // transfer the output nfts to the caller
446:        for (uint256 i = 0; i < outputTokenIds.length; i++) {
447:            ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
448:        }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..859111e 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -438,13 +438,51 @@ contract PrivatePool is ERC721TokenReceiver {
         }

         // transfer the input nfts from the caller
-        for (uint256 i = 0; i < inputTokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
+        address _nft = nft;
+        assembly {
+            if mload(inputTokenIds) {
+                let memptr := mload(0x40)
+                let end := add(add(inputTokenIds, 0x20), mul(0x20, mload(inputTokenIds)))
+                let i := add(inputTokenIds, 0x20)
+                mstore(0x00, 0x42842e0e)
+                mstore(0x20, caller())
+                mstore(0x40, address())
+                for {} 1 {} {
+                    mstore(0x60, mload(i))
+                    let success := call(gas(), _nft, 0x00, 0x1c, 0x64, 0x00, 0x00)
+                    if iszero(success) {
+                        revert(0, 0)
+                    }
+                    i := add(i, 0x20)
+                    if iszero(lt(i, end)) { break }
+                }
+                mstore(0x40, memptr)
+                mstore(0x60, 0x00)
+            }
+
         }

         // transfer the output nfts to the caller
-        for (uint256 i = 0; i < outputTokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
+        assembly {
+            if mload(outputTokenIds) {
+                let memptr := mload(0x40)
+                let end := add(add(outputTokenIds, 0x20), mul(0x20, mload(outputTokenIds)))
+                let i := add(outputTokenIds, 0x20)
+                mstore(0x00, 0x42842e0e)
+                mstore(0x20, address())
+                mstore(0x40, caller())
+                for {} 1 {} {
+                    mstore(0x60, mload(i))
+                    let success := call(gas(), _nft, 0x00, 0x1c, 0x64, 0x00, 0x00)
+                    if iszero(success) {
+                        revert(0, 0)
+                    }
+                    i := add(i, 0x20)
+                    if iszero(lt(i, end)) { break }
+                }
+                mstore(0x40, memptr)
+                mstore(0x60, 0x00)
+            }
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L496-L498

**Note: `PrivatePool.deposit` is fuzzed, which results in inconsistent gas usage during tests. This is why `EthRouter.deposit` is benchmarked instead**.

*Gas Savings for `EthRouter.deposit`, obtained via protocol's tests: Avg 232 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  2371    |  114072  |  29900  |    4     |
| After  |  2371    |  113145  |  29668  |    4     |

```solidity
File: src/PrivatePool.sol
496:        for (uint256 i = 0; i < tokenIds.length; i++) {
497:            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
498:        }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..b22c813 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -493,8 +493,27 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Interactions ~~~ //

         // transfer the nfts from the caller
-        for (uint256 i = 0; i < tokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
+        address _nft = nft;
+        assembly {
+            if tokenIds.length {
+                let memptr := mload(0x40)
+                let end := add(tokenIds.offset, mul(0x20, tokenIds.length))
+                let i := tokenIds.offset
+                mstore(0x00, 0x42842e0e)
+                mstore(0x20, caller())
+                mstore(0x40, address())
+                for {} 1 {} {
+                    mstore(0x60, calldataload(i))
+                    let success := call(gas(), _nft, 0x00, 0x1c, 0x64, 0x00, 0x00)
+                    if iszero(success) {
+                        revert(0, 0)
+                    }
+                    i := add(i, 0x20)
+                    if iszero(lt(i, end)) { break }
+                }
+                mstore(0x40, memptr)
+                mstore(0x60, 0x00)
+            }
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L518-L520

*Gas Savings for `PrivatePool.withdraw`, obtained via protocol's tests: Avg 985 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  81842   |  81842   |  62023  |    5     |
| After  |  80547   |  80547   |  61038  |    5     |

```solidity
File: src/PrivatePool.sol
518:        for (uint256 i = 0; i < tokenIds.length; i++) {
519:            ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
520:        }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..4211c9e 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -515,8 +515,26 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Interactions ~~~ //

         // transfer the nfts to the caller
-        for (uint256 i = 0; i < tokenIds.length; i++) {
-            ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
+        assembly {
+            if tokenIds.length {
+                let memptr := mload(0x40)
+                let end := add(tokenIds.offset, mul(0x20, tokenIds.length))
+                let i := tokenIds.offset
+                mstore(0x00, 0x42842e0e)
+                mstore(0x20, address())
+                mstore(0x40, caller())
+                for {} 1 {} {
+                    mstore(0x60, calldataload(i))
+                    let success := call(gas(), calldataload(0x04), 0x00, 0x1c, 0x64, 0x00, 0x00)
+                    if iszero(success) {
+                        revert(0, 0)
+                    }
+                    i := add(i, 0x20)
+                    if iszero(lt(i, end)) { break }
+                }
+                mstore(0x40, memptr)
+                mstore(0x60, 0x00)
+            }
         }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L673-L679

**Note: `PrivatePool.sumWeightsAndValidateProof` is called by all the functions below**.

*Gas Savings for `PrivatePool.buy`, obtained via protocol's tests: Avg 33 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  74037   |  158864  |  70884  |    24    |
| After  |  74037   |  158864  |  70851  |    24    |

*Gas Savings for `PrivatePool.sell`, obtained via protocol's tests: Avg 74 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  50139   |  170448  |  81969  |    25    |
| After  |  50139   |  170448  |  81895  |    25    |

*Gas Savings for `PrivatePool.change`, obtained via protocol's tests: Avg 157 gas*

|        |    Med   |    Max   |   Avg   | # calls  |
| ------ | -------- | -------- | ------- | -------- |
| Before |  116432  |  142008  |  82138  |    17    |
| After  |  116432  |  142008  |  81981  |    17    |

```solidity
File: src/PrivatePool.sol
673:        for (uint256 i = 0; i < tokenIds.length; i++) {
674:            // create the leaf for the merkle proof
675:            leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
676:
677:            // sum each token weight
678:            sum += tokenWeights[i];
679:        }
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..7d63e8e 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -670,12 +670,25 @@ contract PrivatePool is ERC721TokenReceiver {

         uint256 sum;
         bytes32[] memory leafs = new bytes32[](tokenIds.length);
-        for (uint256 i = 0; i < tokenIds.length; i++) {
-            // create the leaf for the merkle proof
-            leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
-
-            // sum each token weight
-            sum += tokenWeights[i];
+        assembly {
+            if mload(tokenIds) {
+                let end := add(add(tokenIds, 0x20), mul(0x20, mload(tokenIds)))
+                let i := add(tokenIds, 0x20)
+                let j := add(tokenWeights, 0x20)
+                let k := add(leafs, 0x20)
+                for {} 1 {} {
+                    mstore(0x00, mload(i))
+                    mstore(0x20, mload(j))
+                    mstore(0x00, keccak256(0x00, 0x40))
+                    let hash := keccak256(0x00, 0x20)
+                    mstore(k, hash)
+                    sum := add(sum, mload(j))
+                    i := add(i, 0x20)
+                    j := add(j, 0x20)
+                    k := add(k, 0x20)
+                    if iszero(lt(i, end)) { break }
+                }
+            }
         }
```
## `GasReport` output, with all optimizations applied
```js
| src/EthRouter.sol:EthRouter contract |                 |        |        |        |         |
|--------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                      | Deployment Size |        |        |        |         |
| 1415709                              | 7247            |        |        |        |         |
| Function Name                        | min             | avg    | median | max    | # calls |
| buy                                  | 658             | 190464 | 179510 | 374824 | 7       |
| change                               | 576             | 202568 | 265225 | 279247 | 4       |
| deposit                              | 773             | 29393  | 2361   | 112078 | 4       |
| onERC721Received                     | 698             | 698    | 698    | 698    | 92      |
| receive                              | 55              | 55     | 55     | 55     | 16      |
| sell                                 | 732             | 223981 | 209434 | 388776 | 7       |


| src/Factory.sol:Factory contract |                 |        |        |        |         |
|----------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                  | Deployment Size |        |        |        |         |
| 1403424                          | 7270            |        |        |        |         |
| Function Name                    | min             | avg    | median | max    | # calls |
| create                           | 1641            | 148672 | 161112 | 243941 | 35      |
| ownerOf                          | 0               | 20     | 0      | 622    | 30      |
| predictPoolDeploymentAddress     | 2868            | 2868   | 2868   | 2868   | 1       |
| privatePoolImplementation        | 403             | 403    | 403    | 403    | 1       |
| privatePoolMetadata              | 426             | 426    | 426    | 426    | 1       |
| protocolFeeRate                  | 419             | 1332   | 419    | 2419   | 127     |
| receive                          | 55              | 55     | 55     | 55     | 3       |
| setPrivatePoolImplementation     | 22744           | 22744  | 22744  | 22744  | 133     |
| setPrivatePoolMetadata           | 2599            | 22357  | 22634  | 22634  | 127     |
| setProtocolFeeRate               | 7521            | 9678   | 7521   | 22621  | 7       |
| tokenURI                         | 367468          | 367468 | 367468 | 367468 | 1       |
| withdraw                         | 2729            | 12149  | 11128  | 23611  | 4       |


| src/PrivatePool.sol:PrivatePool contract |                 |       |        |        |         |
|------------------------------------------|-----------------|-------|--------|--------|---------|
| Deployment Cost                          | Deployment Size |       |        |        |         |
| 3097024                                  | 15926           |       |        |        |         |
| Function Name                            | min             | avg   | median | max    | # calls |
| baseToken                                | 404             | 904   | 404    | 2404   | 12      |
| buy                                      | 1123            | 69821 | 73315  | 157662 | 24      |
| buyQuote                                 | 2282            | 5817  | 4282   | 10782  | 28      |
| change                                   | 4723            | 78083 | 111589 | 134100 | 17      |
| changeFee                                | 366             | 366   | 366    | 366    | 2       |
| changeFeeQuote                           | 3070            | 9202  | 10796  | 10796  | 18      |
| deposit                                  | 793             | 62671 | 13721  | 357344 | 8       |
| execute                                  | 3615            | 18550 | 17976  | 35939  | 6       |
| factory                                  | 261             | 261   | 261    | 261    | 2       |
| feeRate                                  | 375             | 375   | 375    | 375    | 5       |
| flashFee                                 | 539             | 1872  | 2539   | 2539   | 3       |
| flashFeeToken                            | 419             | 819   | 419    | 2419   | 5       |
| flashLoan                                | 62820           | 82915 | 82915  | 103011 | 2       |
| initialize                               | 1205            | 62085 | 55637  | 95437  | 147     |
| initialized                              | 418             | 418   | 418    | 418    | 1       |
| merkleRoot                               | 385             | 385   | 385    | 385    | 3       |
| nft                                      | 383             | 383   | 383    | 383    | 6       |
| onERC721Received                         | 840             | 840   | 840    | 840    | 151     |
| payRoyalties                             | 393             | 393   | 393    | 393    | 3       |
| price                                    | 1185            | 3866  | 5185   | 5185   | 7       |
| receive                                  | 55              | 55    | 55     | 55     | 33      |
| sell                                     | 6077            | 81284 | 49660  | 169661 | 25      |
| sellQuote                                | 2407            | 5657  | 4407   | 10907  | 22      |
| setFeeRate                               | 3384            | 6451  | 6459   | 9503   | 4       |
| setMerkleRoot                            | 9425            | 9425  | 9425   | 9425   | 2       |
| setPayRoyalties                          | 3406            | 6164  | 6704   | 9504   | 5       |
| setUseStolenNftOracle                    | 3428            | 5628  | 6729   | 6729   | 3       |
| setVirtualReserves                       | 3568            | 7639  | 8530   | 9930   | 4       |
| useStolenNftOracle                       | 417             | 417   | 417    | 417    | 3       |
| virtualBaseTokenReserves                 | 470             | 470   | 470    | 470    | 8       |
| virtualNftReserves                       | 465             | 465   | 465    | 465    | 8       |
| withdraw                                 | 3774            | 61038 | 80547  | 80547  | 5       |
```