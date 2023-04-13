### EthRouter.sol

1. In EthRouter.sol, the `buy()` function does not check if array is too large to be executed in a single transaction. This may result in an out-of-gas error if the array is large.
    [Link to code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L99)

2. In EthRouter.sol, the `buy()` function calls the `nftBuy()` function of the `Pair` contract, but it does not check the returned value. If the `nftBuy()` function reverts for some real, the `buy()` function will continue executing and may result in unexpected way.
    [Link to code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L109)

3. In EthRouter.sol, the `buy()` function calculates the royalty fee and recipient for each token separately, which may result in higher gas cost than necessary. It would be more efficient to calculate the royalty fee and recipient for all tokens at once, and then transfer the total royalty fee to the royalty recipient.
    [Link to code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L116)

4. In EthRouter.sol, the `buy()` function does not check if the `salePrice` is greater than or equal to the royalty fee. If the salePrice is smaller than the royalty fee, the transfer function will revert and the transaction will fail.
    [Link to code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L121)
    Such type of check would work.
    ```
    require(salePrice >= royaltyFee, "Sale price must be greater than or equal to royalty fee");
    ```

### Factory.sol

5. In Factory.sol, the `create()` function, there is no check for whether the NFTs that are being deposited to the pool have been approved for transfer by the caller. This could lead to loss of funds if the caller forgets to approve the transfer before calling this function.
    [Link to code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L120)

6. In Factory.sol, the `create()` function takes in a `tokenIds` parameter, which is an array of token ids to deposit to the pool. However, there is no check to ensure that the caller actually owns these tokens. This could lead to an attacker being able to create a private pool with tokens that they don't actually own.
    [Link to code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L82)

7. In Factory.sol, the `create()` function accepts multiple arguments some of which are complex data types like arrays & structs. However, there is no input validation being performed on any of these arguments. This leaves the contract open to potential attacks where malicious actors could pass invalid or unexpected input data to the function.
    [Link to code](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L71)

### PrivatePoolMetadata.sol

8. In PrivatePoolMetadata.sol, the smart contract functions does not validate input parameters like `tokenId` (even for `public view`). It makes it possible for an attacker to pass in a malicious input that can cause the contract to behave unexpectedly.

### PrivatePool.sol

9. In PrivatePool.sol, pretty much all the `set` functions need additional validation (non-zero addresses, non-zero uints) along with `onlyOwner` modifier just to be safe and avoid unexpected issues.

10. In PrivatePool.sol, `initialize()` function needs validation for all the inputs to avoid unexpected issues.