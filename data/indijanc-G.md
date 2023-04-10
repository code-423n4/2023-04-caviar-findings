## Use unchecked{} when there's logically no way of under- or overflow
There's a lot of places in the contracts where an unchecked block can be used to reduce gas cost. Pretty much all the for loops can have the unchecked index increment as there's no way of under- or overflow. Testing in Remix the cost savings between `i++` and `unchecked{i++}` is 120 gas. With so many for loop iteration this can save up a considerable amount of gas. See the following diff where the unchecked block could be added and see the audit comments for the rationale. See bottom tables to review gas savings as recorded by the project test suite.

#### Diff
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..6e99563 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -103,7 +103,8 @@ contract EthRouter is ERC721TokenReceiver {
         }
 
         // loop through and execute the the buys
-        for (uint256 i = 0; i < buys.length; i++) {
+        // @audit-info - no way of providing so many buys that this would overflow before getting out of gas error (same for inner loops)
+        for (uint256 i = 0; i < buys.length; i = uncheckedInc(i)) {
             if (buys[i].isPublicPool) {
                 // execute the buy against a public pool
                 uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(
@@ -113,7 +114,7 @@ contract EthRouter is ERC721TokenReceiver {
                 // pay the royalties if buyer has opted-in
                 if (payRoyalties) {
                     uint256 salePrice = inputAmount / buys[i].tokenIds.length;
-                    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
+                    for (uint256 j = 0; j < buys[i].tokenIds.length; j = uncheckedInc(j)) {
                         // get the royalty fee and recipient
                         (uint256 royaltyFee, address royaltyRecipient) =
                             getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);
@@ -131,7 +132,7 @@ contract EthRouter is ERC721TokenReceiver {
                 );
             }
 
-            for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
+            for (uint256 j = 0; j < buys[i].tokenIds.length; j = uncheckedInc(j)) {
                 // transfer the NFT to the caller
                 ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
             }
@@ -156,9 +157,10 @@ contract EthRouter is ERC721TokenReceiver {
         }
 
         // loop through and execute the sells
-        for (uint256 i = 0; i < sells.length; i++) {
+        // @audit-info - no way of providing so many sells that this would overflow before getting out of gas error (same for inner loops)
+        for (uint256 i = 0; i < sells.length; i = uncheckedInc(i)) {
             // transfer the NFTs into the router from the caller
-            for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
+            for (uint256 j = 0; j < sells[i].tokenIds.length; j = uncheckedInc(j)) {
                 ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
             }
 
@@ -180,7 +182,7 @@ contract EthRouter is ERC721TokenReceiver {
                 // pay the royalties if seller has opted-in
                 if (payRoyalties) {
                     uint256 salePrice = outputAmount / sells[i].tokenIds.length;
-                    for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
+                    for (uint256 j = 0; j < sells[i].tokenIds.length; j = uncheckedInc(j)) {
                         // get the royalty fee and recipient
                         (uint256 royaltyFee, address royaltyRecipient) =
                             getRoyalty(sells[i].nft, sells[i].tokenIds[j], salePrice);
@@ -236,8 +238,10 @@ contract EthRouter is ERC721TokenReceiver {
         }
 
         // transfer NFTs from caller
-        for (uint256 i = 0; i < tokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
+        unchecked {// @audit-info - no way of providing so many tokenIds that this would overflow before getting out of gas error
+            for (uint256 i = 0; i < tokenIds.length; i++) {
+                ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
+            }
         }
 
         // approve pair to transfer NFTs from router
@@ -258,11 +262,12 @@ contract EthRouter is ERC721TokenReceiver {
         }
 
         // loop through and execute the changes
-        for (uint256 i = 0; i < changes.length; i++) {
+        // @audit-info - no way of providing so many changes that this would overflow before getting out of gas error (same for inner loop)
+        for (uint256 i = 0; i < changes.length; i = uncheckedInc(i)) {
             Change memory _change = changes[i];
 
             // transfer NFTs from caller
-            for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
+            for (uint256 j = 0; j < changes[i].inputTokenIds.length; j = uncheckedInc(j)) {
                 ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);
             }
 
@@ -281,7 +286,7 @@ contract EthRouter is ERC721TokenReceiver {
             );
 
             // transfer NFTs to caller
-            for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
+            for (uint256 j = 0; j < changes[i].outputTokenIds.length; j = uncheckedInc(j)) {
                 ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);
             }
         }
@@ -314,4 +319,7 @@ contract EthRouter is ERC721TokenReceiver {
             if (royaltyFee > salePrice) revert InvalidRoyaltyFee();
         }
     }
+
+    /// @notice Unchecked increment 
+    function uncheckedInc(uint256 index) internal pure returns (uint256) { unchecked { return index + 1; } }
 }
diff --git a/src/Factory.sol b/src/Factory.sol
index 09cbb4e..6cc6ff5 100644
--- a/src/Factory.sol
+++ b/src/Factory.sol
@@ -116,8 +116,10 @@ contract Factory is ERC721, Owned {
         }
 
         // deposit the nfts from the caller into the pool
-        for (uint256 i = 0; i < tokenIds.length; i++) {
-            ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
+        unchecked {// @audit-info - no way of providing so many tokenIds that this would overflow before getting out of gas error
+            for (uint256 i = 0; i < tokenIds.length; i++) {
+                ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
+            }
         }
 
         // emit create event
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..96c433f 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -235,7 +235,8 @@ contract PrivatePool is ERC721TokenReceiver {
         // calculate the sale price (assume it's the same for each NFT even if weights differ)
         uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
         uint256 royaltyFeeAmount = 0;
-        for (uint256 i = 0; i < tokenIds.length; i++) {
+        // @audit-info - no way of providing so many tokenIds that this would overflow before getting out of gas error
+        for (uint256 i = 0; i < tokenIds.length; i = uncheckedInc(i)) {
             // transfer the NFT to the caller
             ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
 
@@ -269,7 +270,8 @@ contract PrivatePool is ERC721TokenReceiver {
         }
 
         if (payRoyalties) {
-            for (uint256 i = 0; i < tokenIds.length; i++) {
+            // @audit-info - no way of providing so many tokenIds that this would overflow before getting out of gas error
+            for (uint256 i = 0; i < tokenIds.length; i = uncheckedInc(i)) {
                 // get the royalty fee for the NFT
                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);
 
@@ -326,7 +328,8 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Interactions ~~~ //
 
         uint256 royaltyFeeAmount = 0;
-        for (uint256 i = 0; i < tokenIds.length; i++) {
+        // @audit-info - no way of providing so many tokenIds that this would overflow before getting out of gas error
+        for (uint256 i = 0; i < tokenIds.length; i = uncheckedInc(i)) {
             // transfer each nft from the caller
             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
 
@@ -432,19 +435,23 @@ contract PrivatePool is ERC721TokenReceiver {
             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
 
             // refund any excess ETH to the caller
-            if (msg.value > feeAmount + protocolFeeAmount) {
-                msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);
+            unchecked { // @audit-info - use unchecked because first if statement in this block already makes sure there's no way of over- or underflow here
+                if (msg.value > feeAmount + protocolFeeAmount) {
+                    msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);
+                }
             }
         }
 
-        // transfer the input nfts from the caller
-        for (uint256 i = 0; i < inputTokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
-        }
+        unchecked {// @audit-info - use unchecked to save gas as it is completely unrealistic uint256 i in for loops could overflow before transactions reverts with out of gas error
+            // transfer the input nfts from the caller
+            for (uint256 i = 0; i < inputTokenIds.length; i++) {
+                ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
+            }
 
-        // transfer the output nfts to the caller
-        for (uint256 i = 0; i < outputTokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
+            // transfer the output nfts to the caller
+            for (uint256 i = 0; i < outputTokenIds.length; i++) {
+                ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
+            }
         }
 
         // emit the change event
@@ -491,10 +498,11 @@ contract PrivatePool is ERC721TokenReceiver {
         }
 
         // ~~~ Interactions ~~~ //
-
-        // transfer the nfts from the caller
-        for (uint256 i = 0; i < tokenIds.length; i++) {
-            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
+        unchecked {// @audit-info - use unchecked to save gas as it is completely unrealistic uint256 i in the for loop could overflow before transactions reverts with out of gas error
+            // transfer the nfts from the caller
+            for (uint256 i = 0; i < tokenIds.length; i++) {
+                ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
+            }
         }
 
         if (baseToken != address(0)) {
@@ -515,8 +523,10 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Interactions ~~~ //
 
         // transfer the nfts to the caller
-        for (uint256 i = 0; i < tokenIds.length; i++) {
-            ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
+        unchecked {// @audit-info - use unchecked to save gas as it is completely unrealistic uint256 i in the for loop could overflow before transactions reverts with out of gas error
+            for (uint256 i = 0; i < tokenIds.length; i++) {
+                ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
+            }
         }
 
         if (token == address(0)) {
@@ -670,7 +680,8 @@ contract PrivatePool is ERC721TokenReceiver {
 
         uint256 sum;
         bytes32[] memory leafs = new bytes32[](tokenIds.length);
-        for (uint256 i = 0; i < tokenIds.length; i++) {
+        // @audit-info - no way of providing so many tokenIds that this would overflow before getting out of gas error
+        for (uint256 i = 0; i < tokenIds.length; i = uncheckedInc(i)) {
             // create the leaf for the merkle proof
             leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
 
@@ -791,4 +802,7 @@ contract PrivatePool is ERC721TokenReceiver {
             if (royaltyFee > salePrice) revert InvalidRoyaltyFee();
         }
     }
+
+    /// @notice Unchecked increment 
+    function uncheckedInc(uint256 index) internal pure returns (uint256) { unchecked { return index + 1; } }
 }
```

#### Gas savings as seen by tests
Displaying only rows where there's a difference in gas cost.

Original tests:
| src/EthRouter.sol:EthRouter contract |                 |        |        |        |         |
|--------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                      | Deployment Size |        |        |        |         |
| 2022979                              | 10289           |        |        |        |         |
| Function Name                        | min             | avg    | median | max    | # calls |
| buy                                  | 670             | 199750 | 187054 | 397581 | 7       |
| change                               | 588             | 217295 | 284857 | 298879 | 4       |
| deposit                              | 785             | 29900  | 2371   | 114072 | 4       |
| sell                                 | 744             | 232102 | 217300 | 402940 | 7       |

| src/Factory.sol:Factory contract |                 |        |        |        |         |
|----------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                  | Deployment Size |        |        |        |         |
| 1433451                          | 7420            |        |        |        |         |
| Function Name                    | min             | avg    | median | max    | # calls |
| create                           | 1641            | 148801 | 161124 | 245619 | 35      |

| src/PrivatePool.sol:PrivatePool contract |                 |        |        |         |         |
|------------------------------------------|-----------------|--------|--------|---------|---------|
| Deployment Cost                          | Deployment Size |        |        |         |         |
| 3248407                                  | 16682           |        |        |         |         |
| Function Name                            | min             | avg    | median | max     | # calls |
| buy                                      | 5199            | 70884  | 74037  | 158864  | 24      |
| change                                   | 5169            | 82138  | 116432 | 142008  | 17      |
| deposit                                  | 890             | 210885 | 13909  | 1539874 | 8       |
| sell                                     | 7263            | 81969  | 50139  | 170448  | 25      |
| withdraw                                 | 3774            | 62023  | 81842  | 81842   | 5       |

After applying diff:
| src/EthRouter.sol:EthRouter contract |                 |        |        |        |         |
|--------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                      | Deployment Size |        |        |        |         |
| 2001557                              | 10182           |        |        |        |         |
| Function Name                        | min             | avg    | median | max    | # calls |
| buy                                  | 670             | 199218 | 186574 | 396318 | 7       |
| change                               | 588             | 216356 | 283650 | 297537 | 4       |
| deposit                              | 785             | 29841  | 2371   | 113836 | 4       |
| sell                                 | 744             | 231611 | 216820 | 402034 | 7       |

| src/Factory.sol:Factory contract |                 |        |        |        |         |
|----------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                  | Deployment Size |        |        |        |         |
| 1424244                          | 7374            |        |        |        |         |
| Function Name                    | min             | avg    | median | max    | # calls |
| create                           | 1641            | 148785 | 161124 | 245383 | 35      |

| src/PrivatePool.sol:PrivatePool contract |                 |        |        |        |         |
|------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                          | Deployment Size |        |        |        |         |
| 3226583                                  | 16573           |        |        |        |         |
| Function Name                            | min             | avg    | median | max    | # calls |
| buy                                      | 5131            | 70738  | 73901  | 158447 | 24      |
| change                                   | 5169            | 81803  | 116012 | 141217 | 17      |
| deposit                                  | 890             | 143004 | 13909  | 997100 | 8       |
| sell                                     | 7127            | 81819  | 50030  | 170244 | 25      |
| withdraw                                 | 3774            | 61888  | 81665  | 81665  | 5       |

Gas cost difference:
| src/EthRouter.sol:EthRouter contract |                 |        |        |        |         |
|--------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                      | Deployment Size |        |        |        |         |
| **-21422**                              | **-107**           |        |        |        |         |
| Function Name                        | min             | avg    | median | max    | # calls |
| buy                                  | 0             | **-532** | **-480** | **-1263** | -       |
| change                               | 0             | **-939** | **-1297** | **-1342** | -       |
| deposit                              | 0             | **-59**  | 0   | **-236** | -       |
| sell                                 | 0             | **-491** | **-480** | **-906** | -       |

| src/Factory.sol:Factory contract |                 |        |        |        |         |
|----------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                  | Deployment Size |        |        |        |         |
| **-9207**                          | **-49**            |        |        |        |         |
| Function Name                    | min             | avg    | median | max    | # calls |
| create                           | 0            | **-16** | 0 | **-236** | -      |

| src/PrivatePool.sol:PrivatePool contract |                 |        |        |        |         |
|------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                          | Deployment Size |        |        |        |         |
| **-21824**                                  | **-109**           |        |        |        |         |
| Function Name                            | min             | avg    | median | max    | # calls |
| buy                                      | **-68**            | **-146**  | **-136**  | **-417** | -      |
| change                                   | 0            | **-335**  | **-420** | **-791** | -      |
| deposit                                  | 0             | **-67881**(non-deterministic) | 0  | **-542774**(non-deterministic) | -       |
| sell                                     | **-136**            | **-150**  | **-109**  | **-204** | -      |
| withdraw                                 | 0            | **-135**  | **-177**  | **-177**  | -       |


## Cache indexed element in for loop if used more than once
Whenever accessing indexed element of an array more than once it's worth caching it to reduce gas cost on array lookups. This can be done in the `sell()` and `buy()` functions of `EthRouter` contract and brings considerable gas savings. See the diff to review code changes and the tables at the end to see gas savings as recorded by the project tests.

#### Diff
```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..129c264 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -104,19 +104,20 @@ contract EthRouter is ERC721TokenReceiver {
 
         // loop through and execute the the buys
         for (uint256 i = 0; i < buys.length; i++) {
-            if (buys[i].isPublicPool) {
+            Buy calldata buy = buys[i];
+            if (buy.isPublicPool) {
                 // execute the buy against a public pool
-                uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(
-                    buys[i].tokenIds, buys[i].baseTokenAmount, 0
+                uint256 inputAmount = Pair(buy.pool).nftBuy{value: buy.baseTokenAmount}(
+                    buy.tokenIds, buy.baseTokenAmount, 0
                 );
 
                 // pay the royalties if buyer has opted-in
                 if (payRoyalties) {
-                    uint256 salePrice = inputAmount / buys[i].tokenIds.length;
-                    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
+                    uint256 salePrice = inputAmount / buy.tokenIds.length;
+                    for (uint256 j = 0; j < buy.tokenIds.length; j++) {
                         // get the royalty fee and recipient
                         (uint256 royaltyFee, address royaltyRecipient) =
-                            getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);
+                            getRoyalty(buy.nft, buy.tokenIds[j], salePrice);
 
                         if (royaltyFee > 0) {
                             // transfer the royalty fee to the royalty recipient
@@ -126,12 +127,12 @@ contract EthRouter is ERC721TokenReceiver {
                 }
             } else {
                 // execute the buy against a private pool
-                PrivatePool(buys[i].pool).buy{value: buys[i].baseTokenAmount}(
-                    buys[i].tokenIds, buys[i].tokenWeights, buys[i].proof
+                PrivatePool(buy.pool).buy{value: buy.baseTokenAmount}(
+                    buy.tokenIds, buy.tokenWeights, buy.proof
                 );
             }
 
-            for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
+            for (uint256 j = 0; j < buy.tokenIds.length; j++) {
                 // transfer the NFT to the caller
                 ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
             }
@@ -157,33 +158,34 @@ contract EthRouter is ERC721TokenReceiver {
 
         // loop through and execute the sells
         for (uint256 i = 0; i < sells.length; i++) {
+            Sell calldata sell = sells[i];
             // transfer the NFTs into the router from the caller
-            for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
-                ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
+            for (uint256 j = 0; j < sell.tokenIds.length; j++) {
+                ERC721(sell.nft).safeTransferFrom(msg.sender, address(this), sell.tokenIds[j]);
             }
 
             // approve the pair to transfer NFTs from the router
-            ERC721(sells[i].nft).setApprovalForAll(sells[i].pool, true);
+            ERC721(sell.nft).setApprovalForAll(sell.pool, true);
 
-            if (sells[i].isPublicPool) {
+            if (sell.isPublicPool) {
                 // exceute the sell against a public pool
-                uint256 outputAmount = Pair(sells[i].pool).nftSell(
-                    sells[i].tokenIds,
+                uint256 outputAmount = Pair(sell.pool).nftSell(
+                    sell.tokenIds,
                     0,
                     0,
-                    sells[i].publicPoolProofs,
+                    sell.publicPoolProofs,
                     // ReservoirOracle.Message[] is the exact same as IStolenNftOracle.Message[] and can be
                     // decoded/encoded 1-to-1.
-                    abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))
+                    abi.decode(abi.encode(sell.stolenNftProofs), (ReservoirOracle.Message[]))
                 );
 
                 // pay the royalties if seller has opted-in
                 if (payRoyalties) {
-                    uint256 salePrice = outputAmount / sells[i].tokenIds.length;
-                    for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
+                    uint256 salePrice = outputAmount / sell.tokenIds.length;
+                    for (uint256 j = 0; j < sell.tokenIds.length; j++) {
                         // get the royalty fee and recipient
                         (uint256 royaltyFee, address royaltyRecipient) =
-                            getRoyalty(sells[i].nft, sells[i].tokenIds[j], salePrice);
+                            getRoyalty(sell.nft, sell.tokenIds[j], salePrice);
 
                         if (royaltyFee > 0) {
                             // transfer the royalty fee to the royalty recipient
@@ -193,8 +195,8 @@ contract EthRouter is ERC721TokenReceiver {
                 }
             } else {
                 // execute the sell against a private pool
-                PrivatePool(sells[i].pool).sell(
-                    sells[i].tokenIds, sells[i].tokenWeights, sells[i].proof, sells[i].stolenNftProofs
+                PrivatePool(sell.pool).sell(
+                    sell.tokenIds, sell.tokenWeights, sell.proof, sell.stolenNftProofs
                 );
             }
         }
```


#### Gas savings as seen by tests
Displaying only rows where there's a difference in gas cost.

Original tests:
| src/EthRouter.sol:EthRouter contract |                 |        |        |        |         |
|--------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                      | Deployment Size |        |        |        |         |
| 2022979                              | 10289           |        |        |        |         |
| Function Name                        | min             | avg    | median | max    | # calls |
| buy                                  | 670             | 199750 | 187054 | 397581 | 7       |
| sell                                 | 744             | 232102 | 217300 | 402940 | 7       |

After applying diff:
| src/EthRouter.sol:EthRouter contract |                 |        |        |        |         |
|--------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                      | Deployment Size |        |        |        |         |
| 1788921                              | 9114            |        |        |        |         |
| Function Name                        | min             | avg    | median | max    | # calls |
| buy                                  | 670             | 197227 | 185094 | 390967 | 7       |
| sell                                 | 744             | 228397 | 213842 | 395898 | 7       |

Gas cost difference:
| src/EthRouter.sol:EthRouter contract |                 |        |        |        |         |
|--------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                      | Deployment Size |        |        |        |         |
| **-234058**                              | **-1175**            |        |        |        |         |
| Function Name                        | min             | avg    | median | max    | # calls |
| buy                                  | 0             | **-2523** | **-1960** | **-6614** | -       |
| sell                                 | 0             | **-3705** | **-3458** | **-7042** | -       |