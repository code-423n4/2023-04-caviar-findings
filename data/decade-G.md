#### 1. No need for arguments in flashFee function:
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L750-L752
```
   function flashFee(address, uint256) public view returns (uint256) {
        return changeFee;
    }
```
___
#### 2. Recursive calculation in _buy_ function can be avoided. 
 (netInputAmount - feeAmount - protocolFeeAmount) can be stored in one variable instead of calculating recursively. 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L229-L236
```
@@ -227,13 +227,14 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Effects ~~~ //

         // update the virtual reserves
-        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
+        uint addReserves = netInputAmount - feeAmount - protocolFeeAmount;
+        virtualBaseTokenReserves += uint128(addReserves);
         virtualNftReserves -= uint128(weightSum);

         // ~~~ Interactions ~~~ //

         // calculate the sale price (assume it's the same for each NFT even if weights differ)
-        uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
+        uint256 salePrice = (addReserves) / tokenIds.length;
         uint256 royaltyFeeAmount = 0;
         for (uint256 i = 0; i < tokenIds.length; i++) {
             // transfer the NFT to the caller
```
---
#### 3. Static statement can be put outside FOR loop. 
`salePrice` calculated in `sell()` function can be put outside for loop instead of calculating on every iteration. 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L335

```
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
+        uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;
         uint256 royaltyFeeAmount = 0;
         for (uint256 i = 0; i < tokenIds.length; i++) {
             // transfer each nft from the caller
@@ -332,7 +333,6 @@ contract PrivatePool is ERC721TokenReceiver {
 
             if (payRoyalties) {
                 // calculate the sale price (assume it's the same for each NFT even if weights differ)
-                uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;
 
                 // get the royalty fee for the NFT
                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);
```
