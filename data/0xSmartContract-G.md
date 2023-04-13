### Gas Optimizations List
| Number | Optimization Details | Context |
|:--:|:-------| :-----:|
| [G-01] |Structs can be packed into fewer storage slots| 2 |
| [G-02] |Function Calls in a Loop Can Cause Denial of Service (DOS) and out of gas due to not checking the Array length| 8 |
| [G-03] |Missing `zero-address` check in `constructor`|2 |
| [G-04] |Duplicated require()/if() checks should be refactored to a modifier or function| 7 |
| [G-05] |if () / require() statements that check input arguments should be at the top of the functions |2|
| [G-06] |Use ``calldata`` instead of ``memory`` for function arguments that do not get mutated|11 |
| [G-07] |Use assembly to check for `address(0)` |12 |
| [G-08] | `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables |4 |
| [G-09] | Change ``public`` function visibility to ``external`` | 17 |
| [G-10] |Use ``assembly`` to write _address storage values_ | 9 |
| [G-11] | Setting the _constructor_ to `payable`|3 |
| [G-12] |Empty blocks should be removed or emit something | 4 |
| [G-13] |Functions guaranteed to _revert_ when callled by normal users can be marked `payable`  | 11 |
| [G-14] |Use nested if and, avoid multiple check combinationsd|10 |
| [G-15] |Optimize names to save gas|  |

Total 15 issues

### [G-01] Structs can be packed into fewer storage slots

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, this saves gas when writing to storage ~20000 gas.

2 results - 2 files:

|Title | Total Gas Saved | Instance|
|:--:|:-------:|:--:|
|Struct packed | 4k | 2 |

1. The ``Buy`` structure can be packaged as suggested below  (from 7 slots to 6 slots)

    **1 slot win: 1 * 2k  =  2k gas saved**

```solidity
src\EthRouter.sol:

  48:     struct Buy {
  49:         address payable pool;
  50:         address nft;
  51:         uint256[] tokenIds;
  52:         uint256[] tokenWeights;
  53:         PrivatePool.MerkleMultiProof proof;
  54:         uint256 baseTokenAmount;
  55:         bool isPublicPool;
  56:     }

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L48-L56


```diff
src\EthRouter.sol:

  48:     struct Buy {
  49:         address payable pool;
  50:         address nft;
+ 55:         bool isPublicPool;
  51:         uint256[] tokenIds;
  52:         uint256[] tokenWeights;
  53:         PrivatePool.MerkleMultiProof proof;
  54:         uint256 baseTokenAmount;
- 55:         bool isPublicPool;
  56:     }

```

2. The ``Sell`` structure can be packaged as suggested below. (from 6 slots to 5 slots)

    **1 slot win: 1 * 2k  =  2k gas saved**

```solidity
src\EthRouter.sol:

  58:     struct Sell {
  59:         address payable pool;
  60:         address nft;
  61:         uint256[] tokenIds;
  62:         uint256[] tokenWeights;
  63:         PrivatePool.MerkleMultiProof proof;
  64:         IStolenNftOracle.Message[] stolenNftProofs;
  65:         bool isPublicPool;
  66:         bytes32[][] publicPoolProofs;
  67:     }

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L58-L67


```diff
src\EthRouter.sol:

  58:     struct Sell {
  59:         address payable pool;
  60:         address nft;
+ 65:         bool isPublicPool;
  61:         uint256[] tokenIds;
  62:         uint256[] tokenWeights;
  63:         PrivatePool.MerkleMultiProof proof;
  64:         IStolenNftOracle.Message[] stolenNftProofs;
- 65:         bool isPublicPool;
  66:         bytes32[][] publicPoolProofs;
  67:     }

```

### [G-02] Function Calls in a Loop Can Cause Denial of Service (DOS) and out of gas due to not checking the Array length

Function calls made in an infinite loop are prone to gassing with potential resource exhaustion, as they can trap the contract due to gas limitations or failed transactions. Consider limiting the loop length if the array is expected to grow and/or handle a very large list of items to avoid unnecessary waste of gas and denial of service.

8 results - 3 files:
```solidity
src\EthRouter.sol:

  219:     function deposit(
  220:         address payable privatePool,
  221:         address nft,
  222:         uint256[] calldata tokenIds,
  223:         uint256 minPrice,
  224:         uint256 maxPrice,
  225:         uint256 deadline
  226:     ) public payable {
  227:         // check deadline has not passed (if any)
  228:         if (block.timestamp > deadline && deadline != 0) {
  229:             revert DeadlinePassed();
  230:         }
  231: 
  232:         // check pool price is in between min and max
  233:         uint256 price = PrivatePool(privatePool).price();
  234:         if (price > maxPrice || price < minPrice) {
  235:             revert PriceOutOfRange();
  236:         }
  237: 
  238:         // transfer NFTs from caller
  239:         for (uint256 i = 0; i < tokenIds.length; i++) {
  240:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
  241:         }

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L219-L241


```diff
src\EthRouter.sol:

  219:     function deposit(
  220:         address payable privatePool,
  221:         address nft,
  222:         uint256[] calldata tokenIds,
  223:         uint256 minPrice,
  224:         uint256 maxPrice,
  225:         uint256 deadline
  226:     ) public payable {
  227:         // check deadline has not passed (if any)
  228:         if (block.timestamp > deadline && deadline != 0) {
  229:             revert DeadlinePassed();
  230:         }
  231: 
  232:         // check pool price is in between min and max
  233:         uint256 price = PrivatePool(privatePool).price();
  234:         if (price > maxPrice || price < minPrice) {
  235:             revert PriceOutOfRange();
  236:         }
  237:
+              if (tokenIds.length > MAXTOKENIDLENGTH) revert maxLength();
  238:         // transfer NFTs from caller
  239:         for (uint256 i = 0; i < tokenIds.length; i++) {
  240:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
  241:         }

```

```diff
src\EthRouter.sol:

  254:     function change(Change[] calldata changes, uint256 deadline) public payable {
  255:         // check deadline has not passed (if any)
  256:         if (block.timestamp > deadline && deadline != 0) {
  257:             revert DeadlinePassed();
  258:         }
  259: 
  260:         // loop through and execute the changes
+              if (changes.length > MAXCHANGESLENGTH) revert maxLength();
  261:         for (uint256 i = 0; i < changes.length; i++) {
  262:             Change memory _change = changes[i];

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254-L293


```diff
src\Factory.sol:
   71:     function create(
   72:         address _baseToken,
   73:         address _nft,
   74:         uint128 _virtualBaseTokenReserves,
   75:         uint128 _virtualNftReserves,
   76:         uint56 _changeFee,
   77:         uint16 _feeRate,
   78:         bytes32 _merkleRoot,
   79:         bool _useStolenNftOracle,
   80:         bool _payRoyalties,
   81:         bytes32 _salt,
   82:         uint256[] memory tokenIds, // put in memory to avoid stack too deep error
   83:         uint256 baseTokenAmount
   84:     ) public payable returns (PrivatePool privatePool) {
 
  110:         if (_baseToken == address(0)) {
  111:             // transfer eth into the pool if base token is ETH
  112:             address(privatePool).safeTransferETH(baseTokenAmount);
  113:         } else {
  114:             // deposit the base tokens from the caller into the pool
  115:             ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);
  116:         }
  117: 
  118:         // deposit the nfts from the caller into the pool
+              if (tokenIds.length > MAXTOKENIDLENGTH) revert maxLength();  
  119:         for (uint256 i = 0; i < tokenIds.length; i++) {
  120:             ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
  121:         }

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L71-L125

```diff
src\PrivatePool.sol:

  211:     function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
  212:         public
  213:         payable
  214:         returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
  215:     {
  
+              if (tokenIds.length > MAXTOKENIDLENGTH) revert maxLength(); 
  238:         for (uint256 i = 0; i < tokenIds.length; i++) {
  239:             // transfer the NFT to the caller
  240:             ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
  241: 

+              if (tokenIds.length > MAXTOKENIDLENGTH) revert maxLength(); 
  271:         if (payRoyalties) {
  272:             for (uint256 i = 0; i < tokenIds.length; i++) {
  273:                 // get the royalty fee for the NFT
  274:                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L289


```diff
src\PrivatePool.sol:

  301:     function sell(
  302:         uint256[] calldata tokenIds,
  303:         uint256[] calldata tokenWeights,
  304:         MerkleMultiProof calldata proof,
  305:         IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
  306:     ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {


  328:         uint256 royaltyFeeAmount = 0;
+              if (tokenIds.length > MAXTOKENIDLENGTH) revert maxLength();  
  329:         for (uint256 i = 0; i < tokenIds.length; i++) {
  330:             // transfer each nft from the caller
  331:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301-L373

```diff
src\PrivatePool.sol:
  385:     function change(
  386:         uint256[] memory inputTokenIds,
  387:         uint256[] memory inputTokenWeights,
  388:         MerkleMultiProof memory inputProof,
  389:         IStolenNftOracle.Message[] memory stolenNftProofs,
  390:         uint256[] memory outputTokenIds,
  391:         uint256[] memory outputTokenWeights,
  392:         MerkleMultiProof memory outputProof
  393:     ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {

  439: 
  440:         // transfer the input nfts from the caller
+              if (tokenIds.length > MAXTOKENIDLENGTH) revert maxLength();
  441:         for (uint256 i = 0; i < inputTokenIds.length; i++) {
  442:             ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
  443:         }
  444: 
  445:         // transfer the output nfts to the caller
+              if (tokenIds.length > MAXTOKENIDLENGTH) revert maxLength();
  446:         for (uint256 i = 0; i < outputTokenIds.length; i++) {
  447:             ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
  448:         }

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385-L452

```diff
src\PrivatePool.sol:
  484:     function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {
+              if (tokenIds.length > MAXTOKENIDLENGTH) revert maxLength();
  495:         // transfer the nfts from the caller
  496:         for (uint256 i = 0; i < tokenIds.length; i++) {
  497:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
  498:         }

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484-L507


```diff
src\PrivatePool.sol:
  514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
  515:         // ~~~ Interactions ~~~ //
  516: 
  517:         // transfer the nfts to the caller
+              if (tokenIds.length > MAXTOKENIDLENGTH) revert maxLength();  
  518:         for (uint256 i = 0; i < tokenIds.length; i++) {
  519:             ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
  520:         }

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514-L532


###  [G-03] Missing `zero-address` check in `constructor`

There is no zero value check in the constructor. If these state variables are initialized with a possible value of "0", the contract must be redistributed. This possibility means gas consumption. Because of the EVM design, zero value control is the most error-prone value control, as zero value is assigned in case of no value input.

2 results - 2 files:

```solidity
src\EthRouter.sol:

  90:     constructor(address _royaltyRegistry) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90-L92


```solidity
src\PrivatePool.sol:

  143:     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143-L147

### [G-04] Duplicated require()/if() checks should be refactored to a modifier or function

7 results - 3 files:
```solidity
src\EthRouter.sol:

  101:    if (block.timestamp > deadline && deadline != 0) {
  102:             revert DeadlinePassed();
  103:         }

  154:    if (block.timestamp > deadline && deadline != 0) {
  155:             revert DeadlinePassed();
  156:         }

  228:    if (block.timestamp > deadline && deadline != 0) {
  229:             revert DeadlinePassed();
  230:         }

```

```diff
src\EthRouter.sol:

+      modifier checkDeadline () {
+          if (block.timestamp > deadline && deadline != 0) {
+              revert DeadlinePassed();
+          }
+         -;
+      }

```
[EthRouter.sol#L101-L103](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101-L103), [EthRouter.sol#L154-L156](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154-L156), [EthRouter.sol#L228-L230](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228-L230)


```solidity
src\PrivatePool.sol:

  225:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

  397:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

```

```diff
src\PrivatePool.sol:

+      modifier checkEthAmount () {
+          if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
+         -;
+      }

```
[PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225), [PrivatePool.sol#L397](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397)


```solidity
src\PrivatePool.sol:

  277:    if (royaltyFee > 0 && recipient != address(0)) {
  278:        if (baseToken != address(0)) {
  279:            ERC20(baseToken).safeTransfer(recipient, royaltyFee);
  280:        } else {
  281:            recipient.safeTransferETH(royaltyFee);
  282:        }
  283:    }

  
  344:    if (royaltyFee > 0 && recipient != address(0)) {
  345:        if (baseToken != address(0)) {
  346:            ERC20(baseToken).safeTransfer(recipient, royaltyFee);
  347:        } else {
  348:            recipient.safeTransferETH(royaltyFee);
  349:        }
  350:    }

```

```diff
src\PrivatePool.sol:

+      modifier checkEthAmount () {
+          if (royaltyFee > 0 && recipient != address(0)) {
+              if (baseToken != address(0)) {
+                ERC20(baseToken).safeTransfer(recipient, royaltyFee);
+              } else {
+                  recipient.safeTransferETH(royaltyFee);
+              }
+          }
+         -;
+      }

```
[PrivatePool.sol#L277-L283](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277-L283), [PrivatePool.sol#L344-L350](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344-L350)


### [G-05] if () / require() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.

2 results - 2 files:

```solidity
src\PrivatePool.sol:
  211:     function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
  212:         public
  213:         payable
  214:         returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
  215:     {
  216:         // ~~~ Checks ~~~ //
  217: 
  218:         // calculate the sum of weights of the NFTs to buy
  219:         uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
  220: 
  221:         // calculate the required net input amount and fee amount
  222:         (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);
  223: 
  224:         // check that the caller sent 0 ETH if the base token is not ETH
  225:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
  226:
  227:         // ~~~ Effects ~~~ //
  228: 
  229:         // update the virtual reserves
  230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
  231:         virtualNftReserves -= uint128(weightSum);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L231


```diff
  211:     function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
  212:         public
  213:         payable
  214:         returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
  215:     {
  216:         // ~~~ Checks ~~~ //
+ 224:         // check that the caller sent 0 ETH if the base token is not ETH
+ 225:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
  217:
  218:         // calculate the sum of weights of the NFTs to buy
  219:         uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
  220: 
  221:         // calculate the required net input amount and fee amount
  222:         (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);
  223: 
- 224:         // check that the caller sent 0 ETH if the base token is not ETH
- 225:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
  226:
  227:         // ~~~ Effects ~~~ //
  228: 
  229:         // update the virtual reserves
  230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
  231:         virtualNftReserves -= uint128(weightSum);

```


```solidity
src\PrivatePool.sol:

  301:     function sell(
  302:         uint256[] calldata tokenIds,
  303:         uint256[] calldata tokenWeights,
  304:         MerkleMultiProof calldata proof,
  305:         IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
  306:     ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
  307:         // ~~~ Checks ~~~ //
  308: 
  309:         // calculate the sum of weights of the NFTs to sell
  310:         uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
  311: 
  312:         // calculate the net output amount and fee amount
  313:         (netOutputAmount, feeAmount, protocolFeeAmount) = sellQuote(weightSum);
  314: 
  315:         //  check the nfts are not stolen
  316:         if (useStolenNftOracle) {
  317:             IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
  318:         }
  319: 
  320:         // ~~~ Effects ~~~ //
  321: 
  322:         // update the virtual reserves
  323:         virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
  324:         virtualNftReserves += uint128(weightSum);

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301-L324


```diff
src\PrivatePool.sol:

  301:     function sell(
  302:         uint256[] calldata tokenIds,
  303:         uint256[] calldata tokenWeights,
  304:         MerkleMultiProof calldata proof,
  305:         IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
  306:     ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
  307:         // ~~~ Checks ~~~ //
+ 316:         if (useStolenNftOracle) {
+ 317:             IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
+ 318:         }
  308: 
  309:         // calculate the sum of weights of the NFTs to sell
  310:         uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);
  311: 
  312:         // calculate the net output amount and fee amount
  313:         (netOutputAmount, feeAmount, protocolFeeAmount) = sellQuote(weightSum);
  314: 
  315:         //  check the nfts are not stolen
- 316:         if (useStolenNftOracle) {
- 317:             IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
- 318:         }
  319: 
  320:         // ~~~ Effects ~~~ //
  321: 
  322:         // update the virtual reserves
  323:         virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
  324:         virtualNftReserves += uint128(weightSum);

```


### [G-06] Use ``calldata`` instead of ``memory`` for function arguments that do not get mutated

Mark data types as ``calldata`` instead of ``memory`` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as ``calldata``. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies ``memory`` storage.


11 results - 3 files:
```solidity
src\Factory.sol:

  71     function create(
  72         address _baseToken,
  73         address _nft,
  74         uint128 _virtualBaseTokenReserves,
  75         uint128 _virtualNftReserves,
  76         uint56 _changeFee,
  77         uint16 _feeRate,
  78         bytes32 _merkleRoot,
  79         bool _useStolenNftOracle,
  80         bool _payRoyalties,
  81         bytes32 _salt,
  82:        uint256[] memory tokenIds, // put in memory to avoid stack too deep error
  83         uint256 baseTokenAmount
  84     ) public payable returns (PrivatePool privatePool) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L71-L84


```solidity
src\PrivatePool.sol:

  385     function change(
  386:         uint256[] memory inputTokenIds,
  387:         uint256[] memory inputTokenWeights,
  388          MerkleMultiProof memory inputProof,
  389          IStolenNftOracle.Message[] memory stolenNftProofs,
  390:         uint256[] memory outputTokenIds,
  391:         uint256[] memory outputTokenWeights,
  392          MerkleMultiProof memory outputProof
  393     ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {


  459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

  661     function sumWeightsAndValidateProof(
  662:         uint256[] memory tokenIds,
  663:         uint256[] memory tokenWeights,
  664:         MerkleMultiProof memory proof
  665     ) public view returns (uint256) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385-L393


```solidity
src\PrivatePoolMetadata.sol:

  112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112


### [G-07] Use assembly to check for `address(0)`

12 results - 2 files:

```solidity
src\Factory.sol:

  87:         if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
  88              revert PrivatePool.InvalidEthAmount();

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87-L88


```solidity
src\PrivatePool.sol:

  225:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
  
  254:    if (baseToken != address(0)) {

  277:    if (royaltyFee > 0 && recipient != address(0)) {

  278:    if (baseToken != address(0)) {

  344:    if (royaltyFee > 0 && recipient != address(0)) {

  345:        if (baseToken != address(0)) {

  397:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
 
  421:    if (baseToken != address(0)) {


  489:    if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
  490         revert InvalidEthAmount();

  500:    if (baseToken != address(0)) {

  651:    if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225


### [G-08] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

`x += y (x -= y)` costs more gas than `x = x  + y (x = x - y)` for state variables.


4 results - 1 file:
```solidity
src\PrivatePool.sol:

  230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

  231:         virtualNftReserves -= uint128(weightSum);

  323:         virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

  324:         virtualNftReserves += uint128(weightSum);

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230-L231
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323-L324



```diff
src\PrivatePool.sol:

- 324:         virtualNftReserves += uint128(weightSum);
+ 324:         virtualNftReserves = virtualNftReserves  + uint128(weightSum);

```

### [G-09] Change ``public`` function visibility to ``external``

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

17 results - 4 files:
```solidity
src\EthRouter.sol:

  219     function deposit(
  220         address payable privatePool,
  221         address nft,
  222         uint256[] calldata tokenIds,
  223         uint256 minPrice,
  224         uint256 maxPrice,
  225         uint256 deadline
  226:     ) public payable {

  254:     function change(Change[] calldata changes, uint256 deadline) public payable {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L219-L226

```solidity
src\Factory.sol:

  129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

  135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
  
  141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
  
  148:     function withdraw(address token, uint256 amount) public onlyOwner {
  
  168:     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129


```solidity
src\PrivatePool.sol:

  157     function initialize(
  158         address _baseToken,
  159         address _nft,
  160         uint128 _virtualBaseTokenReserves,
  161         uint128 _virtualNftReserves,
  162         uint56 _changeFee,
  163         uint16 _feeRate,
  164         bytes32 _merkleRoot,
  165         bool _useStolenNftOracle,
  166         bool _payRoyalties
  167:     ) public {
  
  301     function sell(
  302         uint256[] calldata tokenIds,
  303         uint256[] calldata tokenWeights,
  304         MerkleMultiProof calldata proof,
  305         IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
  306:     ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {

  
  385     function change(
  386         uint256[] memory inputTokenIds,
  387         uint256[] memory inputTokenWeights,
  388         MerkleMultiProof memory inputProof,
  389         IStolenNftOracle.Message[] memory stolenNftProofs,
  390         uint256[] memory outputTokenIds,
  391         uint256[] memory outputTokenWeights,
  392         MerkleMultiProof memory outputProof
  393:     ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {
  
  459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
  
  484:     function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {
  
  514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
  
  602     function setAllParameters(
  603         uint128 newVirtualBaseTokenReserves,
  604         uint128 newVirtualNftReserves,
  605         bytes32 newMerkleRoot,
  606         uint16 newFeeRate,
  607         bool newUseStolenNftOracle,
  608         bool newPayRoyalties
  609:     ) public {
  
  742:     function price() public view returns (uint256) {
  
  755:     function flashFeeToken() public view returns (address) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157-L167


```solidity
src\PrivatePoolMetadata.sol:

  17:     function tokenURI(uint256 tokenId) public view returns (string memory) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17


### [G-10] Use ``assembly`` to write _address storage values_ 

9 results - 3 files:
```solidity
src\EthRouter.sol:

  90      constructor(address _royaltyRegistry) {
  91:         royaltyRegistry = _royaltyRegistry;
  92      }

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L91


```solidity
src\PrivatePool.sol:

  143     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
  144:          factory = payable(_factory);
  145:          royaltyRegistry = _royaltyRegistry;
  146:          stolenNftOracle = _stolenNftOracle;

  
  157:     function initialize(
  175:         baseToken = _baseToken;
  176:         nft = _nft;

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L144-L146


```solidity
src\Factory.sol:

  129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
  130:         privatePoolMetadata = _privatePoolMetadata;
  131:     }


  135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
  136:         privatePoolImplementation = _privatePoolImplementation;
  137:     }

  141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
  142:         protocolFeeRate = _protocolFeeRate;
  143:     }

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L130
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L136
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L142


**Recommendation Code:**
```diff
src\Factory.sol:

  129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
- 130:         privatePoolMetadata = _privatePoolMetadata;
+                  assembly {                      
+                      sstore(privatePoolMetadata .slot, _privatePoolMetadata)
+                  }                               
  131:     }
          
```


### [G-11] Setting the _constructor_ to `payable`

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of ```msg.value == 0``` and saves ```13 gas``` on deployment with no security risks.

3 results - 3 files:
```solidity
src\EthRouter.sol:
 
  90:     constructor(address _royaltyRegistry) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90


```solidity
src\Factory.sol:
  
  53:     constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L53


```solidity
src\PrivatePool.sol:

  143:     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143


**Recommendation:**
Set the constructor to ```payable```


### [G-12] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){...}else{...} => if(!x){if(y){...}else{...}}). Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

4 results - 3 files:
```solidity
src\EthRouter.sol:
 
  88:     receive() external payable {}

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88

```solidity
src\Factory.sol:

  53:     constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}

  55:     receive() external payable {}

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L53


```solidity
src\PrivatePool.sol:
 
  134:     receive() external payable {}

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L134


### [G-13]  Functions guaranteed to revert_ when callled by normal users can be marked `payable` 

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.


11 results - 2 files:
```solidity
src\Factory.sol:

  129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

  135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

  141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

  148:     function withdraw(address token, uint256 amount) public onlyOwner {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129


```solidity
src\PrivatePool.sol:

  459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

  514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
  
  538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
  
  550:     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
  
  562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {
  
  576:     function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
  
  587:     function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459


**Recommendation:**

Functions guaranteed to revert when called by normal users can be marked payable  (for only ``onlyOwner`` modifiers)

### [G-14] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

10 results - 3 files:

```solidity
src\EthRouter.sol:

  101:    if (block.timestamp > deadline && deadline != 0) {

  154:    if (block.timestamp > deadline && deadline != 0) {

  228:    if (block.timestamp > deadline && deadline != 0) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101


```solidity
src\Factory.sol:

  87:     if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87


```solidity
src\PrivatePool.sol:

  225:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

  277:    if (royaltyFee > 0 && recipient != address(0)) {

  344:    if (royaltyFee > 0 && recipient != address(0)) {

  397:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

  489:    if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
  
  635:    if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225


**Recomendation Code:**
```diff
src\EthRouter.sol:

101:    if (block.timestamp > deadline && deadline != 0) {
+        if (block.timestamp > deadline) {
+            if (deadline != 0) {
+            }
+        }

```


### [G-15] Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via ```Method ID```. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because ```22 gas``` are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 


**Recommendation:** 
Find a lower ```method ID``` name for the most called functions for example Call() vs. Call1() is cheaper by ```22 gas```
For example, the function IDs in the ```Factory.sol``` contract will be the most used; A lower method ID may be given.

Reference: https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92

Factory.sol function names can be named and sorted according to METHOD ID


```js
Sighash   |   Function Signature
========================
3f891189  =>  create(address,address,uint128,uint128,uint56,uint16,bytes32,bool,bool,bytes32,uint256[],uint256)
8325cdcc  =>  setPrivatePoolMetadata(address)
a4680a3a  =>  setPrivatePoolImplementation(address)
2e551c35  =>  setProtocolFeeRate(uint16)
f3fef3a3  =>  withdraw(address,uint256)
c87b56dd  =>  tokenURI(uint256)
16b0274a  =>  predictPoolDeploymentAddress(bytes32)

```