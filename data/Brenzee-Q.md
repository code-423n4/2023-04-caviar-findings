# Summary

## Low Risk issues

|                                                                                                       | Issue                                                                                       | Instances |
| ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- | --------- |
| [L-01](#l-01-privatepooldeposit-function-can-be-used-by-anyone-but-only-owner-can-withdraw-the-funds) | `PrivatePool.deposit` function can be used by anyone, but only owner can withdraw the funds | 1         |
| [L-02](#l-02-array-lengths-are-not-compared-to-each-other)                                            | Array lengths are not compared to each other                                                | 4         |
| [L-03](#l-03-basetoken-with-3-or-less-decimals-will-make-privatepoolchange-unusable)                  | ERC20 tokens with 3 or less decimals will make `PrivatePool.change`                         | 1         |
| [L-04](#l-04-wrong-nft-address-can-be-passed-in-some-of-ethrouter-functions)                          | Wrong `nft` address can be passed in some of `EthRouter` functions                          | 4         |
| [L-05](#l-05-changefee-cannot-be-updated-after-initalizing-privatepool)                               | `changeFee` cannot updated be after initalizing `PrivatePool`                               | 1         |
| [L-06](#l-06-privatepoolflashfee-returns-changefee-value)                                             | `PrivatePool.flashFee` returns `changeFee` value                                            | 1         |
| [L-07](#l-07-zero-values-are-not-checked)                                                             | Zero values are not checked                                                                 | 7         |
| [L-08](#l-08-privatepoolbuy-and-privatepoolsell-functions-are-vulnerable-to-frontrun-attacks)         | `PrivatePool.buy` and `PrivatePool.sell` functions are vulnerable to frontrun attacks       | 2         |

Total - 21 instances over 8 issues

## Non-critical issues

|                                                                                      | Issue                                                                       | Instances |
| ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- | --------- |
| [N-01](#n-01-change-public-to-external-for-functions-that-are-not-called-internally) | Change `public` to `external` for functions, that are not called internally | 22        |

Total - 22 instances over 1 issue

# Low Risk Issues

## \[L-01\] `PrivatePool.deposit` function can be used by anyone, but only owner can withdraw the funds

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484

The answer to the question "What is a shared pool and custom pool?" on the sponsor's website is:
"Shared pools are pools that anyone can deposit into and earn fees from. Everyone's liquidity is combined together and the price range is set to be full. Custom pools are pools that you create yourself where you are the sole owner. You can set much more customised parameters such as the fee rate, price range, and royalty support."

The answer leads me to believe that protocol users should be able to deposit only into shared pools to earn fees. The `PrivatePool.deposit` function allows anyone to deposit with no benefit, but only the owner can withdraw funds out of the `PrivatePool`, which means users would lose their funds.

Consider adding `onlyOwner` modifier on `deposit` function

```solidity
    function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable onlyOwner
```

## \[L-02\] Array lengths are not compared to each other

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L302-L303
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L386-L391
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L662-L663

Multiple functions do not check if the 2 arrays that are passed in the function as parameters are equal in length.

Since all of the functions that do not check if the array lengths are equal use `sumWeightsAndValidateProof`, consider adding `require` statement in `sumWeightsAndValidateProof` that checks if both arrays are equal length.

```solidity
    function sumWeightsAndValidateProof(
        uint256[] memory tokenIds,
        uint256[] memory tokenWeights,
        MerkleMultiProof memory proof
    ) public view returns (uint256) {
        require(inputTokenIds.length == inputTokenWeights.length, "Arrays are not equal length");


        // if the merkle root is not set then set the weight of each nft to be 1e18
        if (merkleRoot == bytes32(0)) {
            return tokenIds.length * 1e18;
        }

        uint256 sum;
        bytes32[] memory leafs = new bytes32[](tokenIds.length);
        for (uint256 i = 0; i < tokenIds.length; i++) {
            // create the leaf for the merkle proof
            leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

            // sum each token weight
            sum += tokenWeights[i];
        }

        // validate that the weights are valid against the merkle proof
        if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)) {
            revert InvalidMerkleProof();
        }

        return sum;
    }

```

## \[L-03\] `baseToken` with 3 or less decimals will make `PrivatePool.change` unusable

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L416
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L733

If a user creates a `PrivatePool` with a `baseToken` that has 3 or less decimals (for example, GUSD which has 2 decimals), the `PrivatePool.change` will be unusable.

```solidity
    function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
        // multiply the changeFee to get the fee per NFT (4 decimals of accuracy)
        // @audit This line will fail with ERC20 tokens that have less than 4 decimals of precision
        uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;
        uint256 feePerNft = changeFee * 10 ** exponent;

        feeAmount = inputAmount * feePerNft / 1e18;
        protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;
    }
```

If these types of tokens are not supported, consider adding a `require` statement that checks if the `baseToken` has more than 4 decimals.

```solidity
    require(ERC20(baseToken).decimals() > 4, "Base token decimals must be greater than 4");
```

## \[L-04\] Wrong `nft` address can be passed in some of `EthRouter` functions

https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/EthRouter.sol#L99
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/EthRouter.sol#L152
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/EthRouter.sol#L221
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/EthRouter.sol#L254

If user passes the wrong `nft` address that is not `nft` address of `privatePool`, `EthRouter.deposit`, `EthRouter.change`, `EthRouter.sell` and `EthRouter.buy` functions will fail.

Consider removing `nft` from function input parameters and use `privatePool.nft()` instead.

Example with `EthRouter.deposit`:

```solidity
    function deposit(
        address payable privatePool,
        uint256[] calldata tokenIds,
        uint256 minPrice,
        uint256 maxPrice,
        uint256 deadline
    ) public payable {
        address nft = PrivatePool(privatePool).nft();
        // check deadline has not passed (if any)
        if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }

        // check pool price is in between min and max
        uint256 price = PrivatePool(privatePool).price();
        if (price > maxPrice || price < minPrice) {
            revert PriceOutOfRange();
        }

        // transfer NFTs from caller
        for (uint256 i = 0; i < tokenIds.length; i++) {
            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
        }

        // approve pair to transfer NFTs from router
        ERC721(nft).setApprovalForAll(privatePool, true);

        // execute deposit
        PrivatePool(privatePool).deposit{value: msg.value}(tokenIds, msg.value);
    }
```

## \[L-05\] `changeFee` cannot be updated after initalizing `PrivatePool`

https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L180

`changeFee` is only set in `initialize` function, which means that `changeFee` cannot be updated afterwards.

Consider adding a function, where `owner` can update `changeFee`.

```solidity
    function setChangeFee(uint56 _changeFee) external onlyOwner {
        require(_changeFee !== 0, "changeFee cannot be 0");
        changeFee = _changeFee;

        emit SetChangeFee(_changeFee);
    }
```

## [L-06] `PrivatePool.flashFee` returns `changeFee` value

https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L751

```solidity
    function flashFee(address, uint256) public view returns (uint256) {
        return changeFee;
    }
```

Function `PrivatePool.flashFee` (that is used in `PrivatePool.flashLoan` function to calculate fee of flash loan) returns `changeFee` value.

Consider renaming `PrivatePool.flashFee` to `PrivatePool.changeFee` to avoid confusion or save `flashFee` as seperate value.

## [L-07] Zero values are not checked

https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L177-L180
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L562-L571
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L538-L545

Multiple values that are set in `PrivatePool.initialize` function and multiple set functions are not checked for zero values.

For example, `_virtualNftReserves` in `PrivatePool.initialize` is not checked if the value is 0. This may cause issues with `PrivatePool.price()` function since division with 0 is not allowed.

```solidity
    function price() public view returns (uint256) {
        // ensure that the exponent is always to 18 decimals of accuracy
        uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
        return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;
    }
```

Consider adding `require` statements for these values:

- `_virtualNftReserves` in `PrivatePool.initialize`
- `newVirtualNftReserves` in `PrivatePool.setVirtualReserves`
- `_virtualBaseTokenReserves` in `PrivatePool.initialize`
- `newVirtualBaseTokenReserves` in `PrivatePool.setVirtualReserves`
- `_changeFee` in `PrivatePool.initialize`
- `_feeRate` in `PrivatePool.initialize`
- `newFeeRate` in `PrivatePool.setFeeRate`

Example for `_virtualNftReserves`

```solidity
    require(_virtualNftReserves != 0, "virtualNftReserves cannot be 0");
```

## [L-08] `PrivatePool.buy` and `PrivatePool.sell` functions are vulnerable to frontrun attacks

https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L211
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L301-L306

Partial user's funds might get stolen. Since sponsor insists to not use `PrivatePool` directly, putting it low category, but it can be easily prevented by adding additional checks in `PrivatePool.buy` and `PrivatePool.sell` functions.

**Example No.1**
Alice wants to buy 3 NFTs. Bob sees this transaction in mempool and frontruns it and buys 2 NFTs right before Alice's transaction, which updates the `virtualBaseTokenReserves` and `virtualNftReserves`. Then Alice's transaciton gets executed and Alice pays much more. After Alice's transaction, Bob sells his NFT

**Example No.2**
Alice has 4 NFTs and decides to sell them. Bob sees that Alice's transaction in mempool and frontruns her transaction, where he sells his NFTs (or buys NFTs from other marketplace and sells them in `PrivatePool`) to update `virtualBaseTokenReserves` and `virtualNftReserves` . After that, Alice's transaction gets executed and she receives less tokens than she thought she would get. Right after Alice's transaction, Bob buys his NFTs back for cheaper (or buys back to exchange it on different NFT marketplace)

Suggesting for `buy` function add additonal parameter `maxAmount` and for `sell` function add parameter `minAmount`

```solidity
    function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof, uint256 maxAmount) {
        ...
        require(netInputAmount <= maxAmount, "netInputAmount is higher than maxAmount");
    }

    function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs, // put in memory to avoid stack too deep error
        uint256 minAmount
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
        ...
        require(netOutputAmount >= minAmount, "netOutputAmount is lower than minAmount");
    }
```

# Non-critical issues

## [N-01] Change `public` to `external` for functions, that are not called internally

https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L157-L167
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L211-L214
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L301-L306
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L385-L393
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L459
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L484
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L514
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L602-L609
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L742
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L755

https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/Factory.sol#L71-L84
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/Factory.sol#L129-L131
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/Factory.sol#L135-L137
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/Factory.sol#L141-L143
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/Factory.sol#L148
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/Factory.sol#L161
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/Factory.sol#L168

https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/EthRouter.sol#L99
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/EthRouter.sol#L152
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/EthRouter.sol#L219-L226
https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/EthRouter.sol#L254

https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePoolMetadata.sol#L17

Functions that are not used internally in the contract should be marked as `external` instead of `public`.

For example, `PrivatePool.withdraw` function is `public`. Change it to `external`

Change `public` to `external`:

```solidity
    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) external onlyOwner
```
