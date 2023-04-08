##

## [L-1] Prevent division by 0

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

```solidity
FILE : 2023-04-caviar/src/EthRouter.sol

115: uint256 salePrice = inputAmount / buys[i].tokenIds.length;
182: uint256 salePrice = outputAmount / sells[i].tokenIds.length;



```
[EthRouter.sol#L115](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L115)