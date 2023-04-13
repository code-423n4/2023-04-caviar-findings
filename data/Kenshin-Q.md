# Royalty receiver can reject, unsupported, or be blacklisted to receive royalty fee token and can result in the whole transaction be reverted.
## Permalinks
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L123
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L190
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L279
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L281
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L346
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L348

## Description
Private pool owner has a privilege to enforce royalty fee, the user can choose for the shared pool. However, the royalty fee receiver is out of the protocol control. It can be a contract that reject or unsupported for ETH receiving. It can be an EOA address that was blacklisted by some ERC20 token, e.g., USDC, USDT, and etc. Every contract of the protocol utilized solmate's SafeTransfer library, meaning that all transfer must be completed successfully, if not, the whole transaction will be reverted. Hence, if there is only a single royalty fee receiver in the transaction who reject or unable to receive the fee, it can cause the whole transaction to be reverted.

## Recommended Mitigation Steps
Royalty fee can be considered a sensitive topic and can be related to legals. Besides, this situation not supposed to be occurred frequently. So, I recommended acknowledging and accepting should have the least consequences. However, if continuity is the top priority than that, the protocol can use try-catch on royalty fee transferring, refund to the owner who are buying or selling it if failed.

---

# Missing event emitting on important/administration changes function
## Permalinks
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L131
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L35-L137
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141-L143

## Description
Important or state changes function should emit events upon successful execution for off-chain tracking.

## Recommended Mitigation Steps
An event of calling critical functions should be generated for security and off-chain monitoring purposes.

---

# Unvalidated for length consistency of arrays
## Permalinks
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L661-L687

## Description
There is no validation of array length to ensure that all arrays are the same size during iteration to prevent index out of bound.

## Recommended Mitigation Steps
It is recommended to validate the array length of all input arrays before processing.

---

# Missing address zero validation for sending royalty fee
## Permalinks
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L123

## Description
There is no validation that the royal fee receiver address must not be `address(0)`, casuing royalty fee could be sent to `address(0)`

## Recommended Mitigation Steps
It is recommended to check and ignore to send royalty fee if the receiver address is `address(0)`.

---

# Missing address zero validation in setter functions
## Permalinks
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L131
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135-L137

## Description
The address can be set to `address(0)` which could result in unexpected behavior or brick the operation. 

## Recommended Mitigation Steps
Setters of address type parameters should include a zero-address validation.

---

# Open-end NFT spending approval
## Permalinks
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L166
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L244
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L270

## Description
EthRouter need to approve the private pool to transfer the NFT into the pool, but never revoke the approval after everything is success. This can let anyone to force the EthRouter to approve any NFT for any address arbitrarily. It may found no impact by itself, however, combining with other flaw might introduce more serious security vulnerability.

## Recommended Mitigation Steps
For the best practice, I recommended to revoke allowance before exiting the function. Or set allowance per token specifically using `ERC721.approve(to, tokenId)` for tighter security.

---

# `protocolFeeRate` can be greater than 100%
## Permalinks
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141-L143

## Description
There is no maximum limit on how `protocolFeeRate` can be set to.

## Recommended Mitigation Steps
Set a maximum limit for `protocolFeeRate`, and should not be more than `10_000` (100%).

---

# `virtualBaseTokenReserves` can be set to 0, resulting in anyone can buy NTFs from the pool for free.
## Permalinks
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L545

## Description
The pool owner can arbitrarily set the `virtualBaseTokenReserves` to 0, resulting the buying price of all NFTs in the pool becomes 0, then anyone can buy NFTs from the pool for free. And also make selling NFTs to the pool always be reverted by arithmetic underflow.

## Recommended Mitigation Steps
It is recommended to put a validation into the code to not allow setting `virtualBaseTokenReserves` value to 0.

---

# `virtualNftReserves` can be set to 0, resulting in unable to buy and NFTs from/to the pool.
## Permalinks
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L545

## Description
The pool owner can arbitrarily set the `virtualNftReserves` to 0, resulting in dividing-by-zero revert.

## Recommended Mitigation Steps
It is recommended to put a validation into the code to not allow setting `virtualNftReserves` value to 0.

---

# Flashloan function don't return excess ETH
## Permalinks
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L623-L654

### Description
The user requires to pay fee during `flashLoan()` call. If the `baseToken` is ERC20, [the exact amount will be taken](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L651) at the end of the function. But if the `baseToken` is ETH, it only check that [`msg.value` much cover the fee](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L635). The flash-borrower will not get excess ETH back when they put ETH more than required. 

## Recommended Mitigation Steps
Return the excess ETH back to the caller before exit the function.