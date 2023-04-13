# `salePrice` can be calculated outside the loop
## Permalinks
* https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L335

## Description
`salePrice` value is the same every round, it can be declared before the for-loop.

## Recommended Mitigation Steps
Move the `salePrice` declaration to be before the for-loop.