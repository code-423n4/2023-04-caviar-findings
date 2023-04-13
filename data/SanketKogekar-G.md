
### EthRouter.sol

1. In EthRouter.sol, the `buy()` function takes an array of `Buy` structs as input, but there is no check to ensure that this array is not empty. If an empty array is passed, it will loop over it, which may result in an unnecessary gas cost.

    [Link to code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L99)


