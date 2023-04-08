## Gas Optimisations

### [G-01]

If statements are made throughout the code base to perform important checks. Instead of using if statements, we can make use of the `require` function to perform the same checks. Modest gas saving that adds up over time.

```
// Example

- if (initialized) revert AlreadyInitialized();

+ require(!initialized, AlreadyInitialized());
```

It will also make the code more readable for the indented code.

Infractions found:
1. PrivatePool.sol
   - ln 170: if (initialized) revert AlreadyInitialized();
   - ln 173: if (_feeRate > 5_000) revert FeeRateTooHigh();
   - ln 226: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
   - ln 263: if (msg.value < netInputAmount) revert InvalidEthAmount();
   - ln 398: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
   - ln 414: if (inputWeightSum < outputWeightSum) revert InsufficientInputWeight();
   - ln 565: if (newFeeRate > 5_000) revert FeeRateTooHigh();
   - ln 630: if (!availableForFlashLoan(token, tokenId)) revert NotAvailableForFlashLoan();
   - ln 636: if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
   - ln 646: if (!success) revert FlashLoanFailed();
   - ln 683: if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)...
   - ln 793: if (royaltyFee > salePrice) revert InvalidRoyaltyFee();

2. EthRouter.sol
   - ln 101: if (block.timestamp > deadline && deadline != 0)... 
   - ln 154: if (block.timestamp > deadline && deadline != 0)...
   - ln 203: if (address(this).balance < minOutputAmount)...
   - ln 234: if (price > maxPrice || price < minPrice)...
   - ln 256: if (block.timestamp > deadline && deadline != 0)...
   - ln 314: if (royaltyFee > salePrice) revert InvalidRoyaltyFee();