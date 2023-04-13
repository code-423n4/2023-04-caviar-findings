 if the buy function, we can store the buys[i].tokenIds.length in variable so that we don't have to call it each time
 It wastes gas to read an array’s length in every iteration of a for loop, even if it is a memory or calldata array: 3 gas per read.

this fix can be implemented in the buy function [EthRouter.sol#L99](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L99) by creating new variable that holds the value of buys[i].tokenIds.length

In the sell function [EthRouter.sol#L152](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L152) of the EthRouter.sol contract by creating new variable that holds the value of sells[i].tokenIds.length

In the function change [EthRouter.sol#L254](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L254) by creating a variable for changes[i].inputTokenIds.length