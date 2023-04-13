# Report

## Gas Optimizations

### GAS-1 Numerous loadings state variables to memory
Loading state variables to memory costs 48952 gas. It is necessary to use a local variable instead of reading a state variable several times in the function, especially in loops. State variables: `baseToken`, `nft`, `payRoyalties`, `factory`, `merkleRoot`.

*Instances (48)*:
```solidity
File: src/PrivatePool.sol

225:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

240:        ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

242:        if (payRoyalties) {

254:    if (baseToken != address(0)) {

256:        ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);

259:        if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

265:        if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

271:    if (payRoyalties) {

278:            if (baseToken != address(0)) {

279:                ERC20(baseToken).safeTransfer(recipient, royaltyFee);

317:        IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);

331:        ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

333:        if (payRoyalties) {

345:            if (baseToken != address(0)) {

346:                ERC20(baseToken).safeTransfer(recipient, royaltyFee);

357:    if (baseToken == address(0)) {

362:        if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

365:            ERC20(baseToken).transfer(msg.sender, netOutputAmount);

368:            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

397:    if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

401:        IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, inputTokenIds, stolenNftProofs);

421:    if (baseToken != address(0)) {

423:        ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);

426:        if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);

432:        if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

442:        ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);

447:        ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);

489:    if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

497:        ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

500:    if (baseToken != address(0)) {

502:        ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);

635:    if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

651:    if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

667:    if (merkleRoot == bytes32(0)) {

682:    if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)) {

```
[Link to code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol)
