## Low severity findings

## Table of contents

|  | Issue | Instances |
| --- | --- | --- |
| [L-01] | mulDivUp always produces a more precise output than division | 1 |
| [L-02] | No 0 length check for tokenIds arrays leads to strange behavior | 9 |

## [L-01] **`mulDivUp`** always produces a more precise output than division

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L719](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L719)

Replace the aforementioned line with the following:

```solidity
FixedPointMathLib.mulDivUp(inputAmount, virtualBaseTokenReserves, (virtualNftReserves+ inputAmount));
```

For smaller quantities, this line of code will consistently generate greater precision. Moreover, the addition of 1 will always favor the pool.


## [L-02] No 0 length check for **`tokenIds`** arrays leads to strange behavior

Calling the following

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L289](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L289)

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301-L373](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301-L373)

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385-L452](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385-L452)

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484-L507](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484-L507)

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514-L532](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514-L532)

with empty arrays will not result in errors and will instead emit their corresponding events with empty payloads.


The following

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L99-L144](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L99-L144)

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L152-L209](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L152-L209)

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254-L293](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254-L293)

will send the entire balance of the **`ethRouter`** contract when called with empty arrays.

[https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L219-L248](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L219-L248)

By creating a private pool and invoking the deposit function mentioned, the following can be utilized to retrieve all of the NFTs owned by the **`ethRouter`** contract. This is made possible due to the specific line in the function that permits the private pool to spend all of the NFTs held by the router.

```solidity
EthRouter.sol

ERC721(nft).setApprovalForAll(privatePool, true);
```

Subsequently, the owner of the pool can transfer the NFTs to any desired address by utilizing the execute function in their private pool contract.