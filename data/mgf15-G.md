
## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Using bools for storage incurs overhead | 3 |
| [GAS-2](#GAS-2) | Cache array length outside of loop | 19 |
| [GAS-3](#GAS-3) | Don't initialize variables with default value | 21 |
| [GAS-4](#GAS-4) | Functions guaranteed to revert when called by normal users can be marked `payable` | 10 |
| [GAS-5](#GAS-5) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 19 |
| [GAS-6](#GAS-6) | Use != 0 instead of > 0 for unsigned integer comparison | 17 |
| [GAS-7](#GAS-7) | <x> += <y> costs more gas than <x> = <x> + <y> for state variables (-= too) | 9 |
| [GAS-8](#GAS-8) | Setting the constructor to payable | 3 |
| [GAS-9](#GAS-9) | Structs can be packed into fewer storage slots | 5 |


### [GAS-1] Using bools for storage incurs overhead
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (3)*:
```solidity
File: 2023-04-caviar/src/PrivatePool.sol

94:     bool public initialized;

97:     bool public payRoyalties;

100:     bool public useStolenNftOracle;

```

### [GAS-2] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (19)*:
```solidity
File: 2023-04-caviar/src/EthRouter.sol

106:         for (uint256 i = 0; i < buys.length; i++) {

116:                     for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

134:             for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

159:         for (uint256 i = 0; i < sells.length; i++) {

161:             for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

183:                     for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

239:         for (uint256 i = 0; i < tokenIds.length; i++) {

261:         for (uint256 i = 0; i < changes.length; i++) {

265:             for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {

284:             for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {

```

```solidity
File: 2023-04-caviar/src/Factory.sol

119:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

238:         for (uint256 i = 0; i < tokenIds.length; i++) {

272:             for (uint256 i = 0; i < tokenIds.length; i++) {

329:         for (uint256 i = 0; i < tokenIds.length; i++) {

441:         for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:         for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:         for (uint256 i = 0; i < tokenIds.length; i++) {

518:         for (uint256 i = 0; i < tokenIds.length; i++) {

673:         for (uint256 i = 0; i < tokenIds.length; i++) {

```


### [GAS-3] Don't initialize variables with default value

*Instances (21)*:
```solidity
File: 2023-04-caviar/src/EthRouter.sol

106:         for (uint256 i = 0; i < buys.length; i++) {

116:                     for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

134:             for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

159:         for (uint256 i = 0; i < sells.length; i++) {

161:             for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

183:                     for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

239:         for (uint256 i = 0; i < tokenIds.length; i++) {

261:         for (uint256 i = 0; i < changes.length; i++) {

265:             for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {

284:             for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {

```

```solidity
File: 2023-04-caviar/src/Factory.sol

119:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

237:         uint256 royaltyFeeAmount = 0;

238:         for (uint256 i = 0; i < tokenIds.length; i++) {

272:             for (uint256 i = 0; i < tokenIds.length; i++) {

328:         uint256 royaltyFeeAmount = 0;

329:         for (uint256 i = 0; i < tokenIds.length; i++) {

441:         for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:         for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:         for (uint256 i = 0; i < tokenIds.length; i++) {

518:         for (uint256 i = 0; i < tokenIds.length; i++) {

673:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

### [GAS-4] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

*Instances (10)*:
```solidity
File: 2023-04-caviar/src/Factory.sol

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:     function withdraw(address token, uint256 amount) public onlyOwner {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550:     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {

576:     function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587:     function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```

```diff
diff --git a/src/Factory.sol b/src/Factory.sol
index 09cbb4e..c8e30f0 100644
--- a/src/Factory.sol
+++ b/src/Factory.sol
@@ -126,26 +126,26 @@ contract Factory is ERC721, Owned {
 
     /// @notice Sets private pool metadata contract.
     /// @param _privatePoolMetadata The private pool metadata contract.
-    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
+    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner payable {
         privatePoolMetadata = _privatePoolMetadata;
     }
 
     /// @notice Sets the private pool implementation contract that newly deployed proxies point to.
     /// @param _privatePoolImplementation The private pool implementation contract.
-    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
+    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner payable {
         privatePoolImplementation = _privatePoolImplementation;
     }
 
     /// @notice Sets the protocol fee that is taken on each buy/sell/change. It's in basis points: 350 = 3.5%.
     /// @param _protocolFeeRate The protocol fee.
-    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
+    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner payable {
         protocolFeeRate = _protocolFeeRate;
     }
 
     /// @notice Withdraws the earned protocol fees.
     /// @param token The token to withdraw.
     /// @param amount The amount to withdraw.
-    function withdraw(address token, uint256 amount) public onlyOwner {
+    function withdraw(address token, uint256 amount) public onlyOwner payable {
         if (token == address(0)) {
             msg.sender.safeTransferETH(amount);
         } else {
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..bb934c5 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -511,7 +511,7 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @param tokenIds The token IDs of the NFTs to withdraw.
     /// @param token The address of the token to withdraw.
     /// @param tokenAmount The amount of tokens to withdraw.
-    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
+    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner payable {
         // ~~~ Interactions ~~~ //
 
         // transfer the nfts to the caller
@@ -535,7 +535,7 @@ contract PrivatePool is ERC721TokenReceiver {
     /// pool. These parameters affect the price and liquidity depth of the pool.
     /// @param newVirtualBaseTokenReserves The new virtual base token reserves.
     /// @param newVirtualNftReserves The new virtual NFT reserves.
-    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
+    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner payable {
         // set the virtual base token reserves and virtual nft reserves
         virtualBaseTokenReserves = newVirtualBaseTokenReserves;
         virtualNftReserves = newVirtualNftReserves;
@@ -547,7 +547,7 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @notice Sets the merkle root. Can only be called by the owner of the pool. The merkle root is used to validate
     /// the NFT weights.
     /// @param newMerkleRoot The new merkle root.
-    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
+    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner payable {
         // set the merkle root
         merkleRoot = newMerkleRoot;
 
@@ -559,7 +559,7 @@ contract PrivatePool is ERC721TokenReceiver {
     /// fee amount when swapping or changing NFTs. The fee rate is in basis points (1/100th of a percent). For example,
     /// 10_000 == 100%, 200 == 2%, 1 == 0.01%.
     /// @param newFeeRate The new fee rate (in basis points)
-    function setFeeRate(uint16 newFeeRate) public onlyOwner {
+    function setFeeRate(uint16 newFeeRate) public onlyOwner payable {
         // check that the fee rate is less than 50%
         if (newFeeRate > 5_000) revert FeeRateTooHigh();
 
@@ -573,7 +573,7 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @notice Sets the whether or not to use the stolen NFT oracle. Can only be called by the owner of the pool. The
     /// stolen NFT oracle is used to check if an NFT is stolen.
     /// @param newUseStolenNftOracle The new use stolen NFT oracle flag.
-    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
+    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner payable {
         // set the use stolen NFT oracle flag
         useStolenNftOracle = newUseStolenNftOracle;
 
@@ -584,7 +584,7 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @notice Sets the pay royalties flag. Can only be called by the owner of the pool. If royalties are enabled then
     /// the pool will pay royalties when buying or selling NFTs.
     /// @param newPayRoyalties The new pay royalties flag.
-    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
+    function setPayRoyalties(bool newPayRoyalties) public onlyOwner payable {
         // set the pay royalties flag
         payRoyalties = newPayRoyalties;


```

### [GAS-5] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (19)*:
```solidity
File: 2023-04-caviar/src/EthRouter.sol

106:         for (uint256 i = 0; i < buys.length; i++) {

116:                     for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

134:             for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

159:         for (uint256 i = 0; i < sells.length; i++) {

161:             for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

183:                     for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

239:         for (uint256 i = 0; i < tokenIds.length; i++) {

261:         for (uint256 i = 0; i < changes.length; i++) {

265:             for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {

284:             for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {

```

```solidity
File: 2023-04-caviar/src/Factory.sol

119:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

238:         for (uint256 i = 0; i < tokenIds.length; i++) {

272:             for (uint256 i = 0; i < tokenIds.length; i++) {

329:         for (uint256 i = 0; i < tokenIds.length; i++) {

441:         for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:         for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:         for (uint256 i = 0; i < tokenIds.length; i++) {

518:         for (uint256 i = 0; i < tokenIds.length; i++) {

673:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

### [GAS-6] Use != 0 instead of > 0 for unsigned integer comparison

*Instances (17)*:
```solidity
File: 2023-04-caviar/src/EthRouter.sol

121:                         if (royaltyFee > 0) {

141:         if (address(this).balance > 0) {

188:                         if (royaltyFee > 0) {

290:         if (address(this).balance > 0) {

```

```solidity
File: 2023-04-caviar/src/Factory.sol

87:         if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

225:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

259:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

265:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

277:                 if (royaltyFee > 0 && recipient != address(0)) {

344:                 if (royaltyFee > 0 && recipient != address(0)) {

362:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

368:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

397:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

426:             if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);

432:             if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

467:         if (returnData.length > 0) {

489:         if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

```

### [GAS‑7] <x> += <y> costs more gas than <x> = <x> + <y> for state variables (-= too)

Using the addition operator instead of plus-equals saves 113 gas. Subtructions act the same way.
*Instances (9)*:

```solidity
File:2023-04-caviar/blob/main/src/PrivatePool.sol

230:        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
231:        virtualNftReserves -= uint128(weightSum);

247:        royaltyFeeAmount += royaltyFee;

252:        netInputAmount += royaltyFeeAmount;

323:        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
324:        virtualNftReserves += uint128(weightSum);

341:        royaltyFeeAmount += royaltyFee;

355:        netOutputAmount -= royaltyFeeAmount;

678:        sum += tokenWeights[i];
```
```diff
diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..9da3b67 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -227,8 +227,8 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Effects ~~~ //
 
         // update the virtual reserves
-        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
-        virtualNftReserves -= uint128(weightSum);
+        virtualBaseTokenReserves = virtualBaseTokenReserves uint128(netInputAmount - feeAmount - protocolFeeAmount);
+        virtualNftReserves = virtualNftReserves - uint128(weightSum);
 
         // ~~~ Interactions ~~~ //
 
@@ -244,12 +244,12 @@ contract PrivatePool is ERC721TokenReceiver {
                 (uint256 royaltyFee,) = _getRoyalty(tokenIds[i], salePrice);
 
                 // add the royalty fee to the total royalty fee amount
-                royaltyFeeAmount += royaltyFee;
+                royaltyFeeAmount = royaltyFeeAmount + royaltyFee;
             }
         }
 
         // add the royalty fee amount to the net input aount
-        netInputAmount += royaltyFeeAmount;
+        netInputAmount = netInputAmount + royaltyFeeAmount;
 
         if (baseToken != address(0)) {
             // transfer the base token from the caller to the contract
@@ -320,8 +320,8 @@ contract PrivatePool is ERC721TokenReceiver {
         // ~~~ Effects ~~~ //
 
         // update the virtual reserves
-        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
-        virtualNftReserves += uint128(weightSum);
+        virtualBaseTokenReserves = virtualBaseTokenReserves - uint128(netOutputAmount + protocolFeeAmount + feeAmount);
+        virtualNftReserves = virtualNftReserves + uint128(weightSum);
 
         // ~~~ Interactions ~~~ //
 
@@ -338,7 +338,7 @@ contract PrivatePool is ERC721TokenReceiver {
                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);
 
                 // tally the royalty fee amount
-                royaltyFeeAmount += royaltyFee;
+                royaltyFeeAmount = royaltyFeeAmount + royaltyFee;
 
                 // transfer the royalty fee to the recipient if it's greater than 0
                 if (royaltyFee > 0 && recipient != address(0)) {
@@ -352,7 +352,7 @@ contract PrivatePool is ERC721TokenReceiver {
         }
 
         // subtract the royalty fee amount from the net output amount
-        netOutputAmount -= royaltyFeeAmount;
+        netOutputAmount = netOutputAmount - royaltyFeeAmount;
 
 @@ -675,7 +675,7 @@ contract PrivatePool is ERC721TokenReceiver {
             leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
 
             // sum each token weight
-            sum += tokenWeights[i];
+            sum = sum + tokenWeights[i];
         }
```
### [GAS-8] Setting the constructor to payable

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

```solidity 
File:2023-04-caviar/blob/main/src/PrivatePool.sol

143:    constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
```

```solidity 
File:2023-04-caviar/blob/main/src/Factory.sol

53:        constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
```

```solidity 
File:2023-04-caviar/blob/main/src/EthRouter.sol

90:    constructor(address _royaltyRegistry) {
```

### [GAS‑9] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

```solidity 
File:2023-04-caviar/blob/main/src/EthRouter.sol

49:    struct Buy {
50:        address payable pool;
51:        address nft;
52:        uint256[] tokenIds;
53:        uint256[] tokenWeights;
54:        PrivatePool.MerkleMultiProof proof;
55:        uint256 baseTokenAmount;
56:        bool isPublicPool;
57:    }

59:    struct Sell {
60:        address payable pool;
61:        address nft;
62:        uint256[] tokenIds;
63:        uint256[] tokenWeights;
64:        PrivatePool.MerkleMultiProof proof;
65:        IStolenNftOracle.Message[] stolenNftProofs;
66:        bool isPublicPool;
67:        bytes32[][] publicPoolProofs;
68:    }

70:    struct Change {
71:        address payable pool;
72:        address nft;
73:        uint256[] inputTokenIds;
74:        uint256[] inputTokenWeights;
75:        PrivatePool.MerkleMultiProof inputProof;
76:        IStolenNftOracle.Message[] stolenNftProofs;
77:        uint256[] outputTokenIds;
78:        uint256[] outputTokenWeights;
79:        PrivatePool.MerkleMultiProof outputProof;
80:    }

```
```solidity 
File:2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol
struct Message {
        bytes32 id;
        bytes payload;
        // The UNIX timestamp when the message was signed by the oracle
        uint256 timestamp;
        // ECDSA signature or EIP-2098 compact signature
        bytes signature;
    }

```

```solidity 
File:2023-04-caviar/blob/main/src/PrivatePool.sol
struct MerkleMultiProof {
        bytes32[] proof;
        bool[] flags;
    }
```
to 

```diff
diff --git a/src/EthRouter.sol b/src/EthRouter.sol
index 125001d..3f96715 100644
--- a/src/EthRouter.sol
+++ b/src/EthRouter.sol
@@ -46,36 +46,36 @@ contract EthRouter is ERC721TokenReceiver {
     using SafeTransferLib for address;
 
     struct Buy {
+        bool isPublicPool;
         address payable pool;
         address nft;
-        uint256[] tokenIds;
-        uint256[] tokenWeights;
         PrivatePool.MerkleMultiProof proof;
         uint256 baseTokenAmount;
-        bool isPublicPool;
+        uint256[] tokenIds;
+        uint256[] tokenWeights;
     }
 
     struct Sell {
+        bool isPublicPool;
         address payable pool;
         address nft;
-        uint256[] tokenIds;
-        uint256[] tokenWeights;
+        bytes32[][] publicPoolProofs;
         PrivatePool.MerkleMultiProof proof;
         IStolenNftOracle.Message[] stolenNftProofs;
-        bool isPublicPool;
-        bytes32[][] publicPoolProofs;
+        uint256[] tokenIds;
+        uint256[] tokenWeights;
     }
 
     struct Change {
         address payable pool;
         address nft;
-        uint256[] inputTokenIds;
-        uint256[] inputTokenWeights;
         PrivatePool.MerkleMultiProof inputProof;
+        PrivatePool.MerkleMultiProof outputProof;
         IStolenNftOracle.Message[] stolenNftProofs;
+        uint256[] inputTokenIds;
+        uint256[] inputTokenWeights;
         uint256[] outputTokenIds;
         uint256[] outputTokenWeights;
-        PrivatePool.MerkleMultiProof outputProof;
     }

diff --git a/src/PrivatePool.sol b/src/PrivatePool.sol
index 75991e1..282a798 100644
--- a/src/PrivatePool.sol
+++ b/src/PrivatePool.sol
@@ -50,8 +50,8 @@ contract PrivatePool is ERC721TokenReceiver {
     /// @notice Merkle proof input for a sparse merkle multi proof. It can be generated with a library like:
     /// https://github.com/OpenZeppelin/merkle-tree#treegetmultiproof
     struct MerkleMultiProof {
-        bytes32[] proof;
         bool[] flags;
+        bytes32[] proof;
     }

diff --git a/src/interfaces/IStolenNftOracle.sol b/src/interfaces/IStolenNftOracle.sol
index 815a356..19d38f4 100644
--- a/src/interfaces/IStolenNftOracle.sol
+++ b/src/interfaces/IStolenNftOracle.sol
@@ -6,10 +6,10 @@ interface IStolenNftOracle {
     struct Message {
         bytes32 id;
         bytes payload;
-        // The UNIX timestamp when the message was signed by the oracle
-        uint256 timestamp;
         // ECDSA signature or EIP-2098 compact signature
         bytes signature;
+        // The UNIX timestamp when the message was signed by the oracle
+        uint256 timestamp;
     }
```