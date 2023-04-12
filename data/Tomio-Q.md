Title: Use `safeTransfer`/`safeTransferFrom` consistently instead of `transfer`/`transferFrom`

Impact:
It is good to add a require() statement that checks the return value of token transfers or to use something like OpenZeppelin’s safeTransfer/safeTransferFrom unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

Proof of Concept:
[Factory.sol#L115](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L115)

Recommended Mitigation Steps
Consider using safeTransfer/safeTransferFrom or require() consistently.

Reference: [here](https://consensys.net/diligence/audits/2021/01/fei-protocol/#unchecked-return-value-for-iweth-transfer-call)
______________________________________________________________________