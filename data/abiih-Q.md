# Report

---

## Low Risks

---

|     | Issue                                                                                                                                                                           |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | [Add non-zero address checks for address arguments in constructors](#usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)                                         |
| 2   | [Fix flash loan fee is unfair to the pool owner as well as to the user ](#fix-flash-loan-fee-is-unfair-to-the-pool-owner-as-well-as-to-the-user)                                |
| 3   | [Royalty receiver will not get correct royalty as saleprice is not calculated properly](#royalty-receiver-will-not-get-correct-royalty-as-saleprice-is-not-calculated-properly) |

---

1. ### Add non-zero address checks for address arguments in constructors

   check address value for zero

   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90

2. ### Fix flash loan fee is unfair to the pool owner as well as to the user

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L632
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L750

```
Here the changeFee is used as the flash loan fee whenever the user initiates the flash loan. This is unfair to both the pool owner as well as to the user.
Different NFT has different rarity as well as value. When the flash loan fee is fixed then pool owner will not be able to collect the proper fee and also the user who want to use the NFT with less rarity and usefulness has to pay the same price as that of the NFT of higher price.
```

3. ### Royalty receiver will not get correct royalty as saleprice is not calculated properly

   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L182

   ```
   uint256 salePrice = outputAmount / sells[i].tokenIds.length;

   ```

   Here the salesprice for an nft is calculated by using the above formula. A user might buy different NFTs in different prices. But when above formula is used for calculating the saleprice of an NFT,the NFT may have higher price or lower price than the actual price the user paid to buy it. Due to this royalty receiver will not be receiving the actual amount that they have to receive.
