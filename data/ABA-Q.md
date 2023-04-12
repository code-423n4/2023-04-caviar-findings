# QA Report for Caviar Private Pools contest

## Overview
During the audit, 3 low, 6 refactoring and 1 non-critical issues were found.

### Low Risk Issues

Total: 10 instances over 3 issues

|#|Issue|Instances|
|-|:-|:-:|
| [L-01] | Input is not checked when setting sensitive addresses across protocol | 4 |
| [L-02] | Any leftover ETH on `EthRouter` can be taken with empty buy/change orders | 2 |
| [L-03] | Precision loss on royalties payments | 4 |

### Refactoring Issues

Total: 10 instances over 6 issues

|#|Issue|Instances|
|-|:-|:-:|
| [R-01]| Events missing key information | 3 |
| [R-02]| Inconsistent checks for `royaltyRecipient` | 2 |
| [R-03]| Reuse `getRoyalty` implementation | 2 |
| [R-04]| Use a constant for the flashLoan callback keccak256 | 1 |
| [R-05]| Combine royalty transfer logic and ERC721 transfer logic in `PrivatePool`'s `buy` function | 1 |
| [R-06]| `buyQuote` reverts with arithmetic underflow instead of meaningful message | 1 |

### Non-critical Issues

Total: 2 instances over 1 issues

|#|Issue|Instances|
|-|:-|:-:|
| [NC-01]| misleading or incorrect documentation | 2 |

#
## Low Risk Issues (3)
#

### [L-01] Input is not checked when setting sensitive addresses across protocol
##### Description

- in `Factory`, when setting the `privatePoolMetadata` or `privatePoolImplementation` addresses, there is no check if the new address is 0 or that it is actually a contract.
- in `EthRouter` when deploying the contract, the `royaltyRegistry` variable is set, but not checked if the provided address is 0 or that it is actually a contract.
- in `PrivatePool` when deploying the contract, the factory, royaltyRegistry and stolenNftOracle are set but not checked if the provided address is 0 or that it is actually a contract.

##### Instances (4)

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L127-L137
```Solidity
    /// @notice Sets private pool metadata contract.
    /// @param _privatePoolMetadata The private pool metadata contract.
    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
        privatePoolMetadata = _privatePoolMetadata;
    }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90-L92

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143-L147

##### Recommendation

Add check for 0 address at minimum. Add check that the provided address is a contract (has code size).

# 

### [L-02] Any leftover ETH on `EthRouter` can be taken with empty buy/change orders
##### Description

In `EthRouter` during a buy or change, ETH is sent to the contract. At the end of the execution of the functions, a check if any leftover ETH remains in the contract and is sent back to the `msg.sender`.
This mechanism ensures that any, normal operation within the contract, will always leave it with a 0 ETH balance.

The issue, of low probability, appears when users mistakenly send ETH to the contract directly without a buy/change operation. This means that on the next buy/change the ETH will be taken by the caller.
Also, as the functions are constructed now, anybody can call them with empty values resulting in the equivalent of calling `msg.sender.safeTransferETH(address(this).balance);` directly.

##### Instances (2)

For buys: https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L99-L144
```Solidity
    function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
        // check that the deadline has not passed (if any)
        if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }


        // loop through and execute the the buys
        for (uint256 i = 0; i < buys.length; i++) {
            // as buys ca be empty this code block is skipped
        }

        // refund any surplus ETH to the caller
        if (address(this).balance > 0) {
            msg.sender.safeTransferETH(address(this).balance);
        }
    }        
```


Identical for change: https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254-L293

##### Recommendation

Since this is by protocol design, there is no way of completely mitigating it. The most that can be suggested to add a check that at least 1 `Buy`/`Change` operation is required and to give a warning in documentation that any ETH sent directly to the router will be lost by MEV opportunists.

#

### [L-03] Precision loss on royalties payments
##### Description

Royalty fee is calculated based on the sale price (`salePrice`) For each instances this sale price is calculated by a division, that results in a minor rounding error loss. Example:

```Solidity
        uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
```

##### Instances (4)

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L236
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L115
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L182

##### Recommendation

Guard the division with a precision multiplier. 

#

#### Refactoring Issues (6)
#


### [R-01] Events missing key information
##### Description

Some events are missing key information when emitted.

##### Instances (3)

- in `Factory` the `Withdraw` event should also contain the current factory owner, as the contract can change owner ([code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L155))
- in `PrivatePool` the `SetVirtualReserves` event should also contain the old values for `virtualBaseTokenReserves` and `virtualNftReserves` ([code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L544))
- in `PrivatePool` the `SetMerkleRoot` event should also contain the old value for `merkleRoot` ([code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L555))


##### Recommendation

Add the missing information.

#

### [R-02] Inconsistent checks for `royaltyRecipient`
##### Description

In the protocol, there are 2 implementations that get the royaltyFee and recipient. Namely `getRoyalty` in `EthRouter` ([code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L301-L316)) and `_getRoyalty` in `PrivatePool` ([code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L778-L794)).

Both have identical outputs. These outputs are then verified in code, but in one case the check is for the existance of the fee and recipient to not be 0 address

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277
```Solidity
    if (royaltyFee > 0 && recipient != address(0))
```
and in other cases only if the fee exists.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L121
```Solidity
    if (royaltyFee > 0) 
```

The extra check for the 0 address should be duplicated in all occurrences.

##### Instances (2)

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L121
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L188

##### Recommendation

Add the recipient 0 address check on all occurrences.

#

### [R-03] Reuse `getRoyalty` implementation

##### Description

In the protocol, there are 2 implementations that get the royaltyFee and recipient. Namely `getRoyalty` in `EthRouter` ([code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L301-L316)) and `_getRoyalty` in `PrivatePool` ([code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L778-L794)).

Both have identical outputs and only differ slightly in implementation. They can be moved to a separate *utils* like library and be then used by both `getRoyalty` and `EthRouter`.

##### Instances (2)

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L301-L316
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L778-L794

##### Recommendation

Move `getRoyalty` logic to a separate contract and reuse it.
Also, in the `PrivatePool` case, the function can also be made public.

#

### [R-04] Use a constant for the flashLoan callback keccak256
##### Description

When executing a `flashLoan` the result from calling `onFlashLoan` is checked, as per standard with the `keccak256("ERC3156FlashBorrower.onFlashLoan")`.
This, however is done on each call, resulting in the same `keccak256` calculation to be done multiple times.


```Solidity
    bool success =
        receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");
```

##### Instances (1)

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L642

##### Recommendation
Precalculate the `keccak256`, save it, and compare with it on each loan, instead of calculating it each time.

```Solidity
    bytes32 public constant CALLBACK_SUCCESS = keccak256("ERC3156FlashBorrower.onFlashLoan");
```

#

### [R-05] Combine royalty transfer logic and ERC721 transfer logic in `PrivatePool`'s `buy` function
##### Description

In `PrivatePool`, in the `buy` function there are 2 set of operations that can be combined, without any reentrancy issues, together:

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238-L249
```Solidity
        for (uint256 i = 0; i < tokenIds.length; i++) {
            // transfer the NFT to the caller
            ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);


            if (payRoyalties) {
                // get the royalty fee for the NFT
                (uint256 royaltyFee,) = _getRoyalty(tokenIds[i], salePrice);


                // add the royalty fee to the total royalty fee amount
                royaltyFeeAmount += royaltyFee;
            }
        }
```
and

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L271-L285
```Solidity
        if (payRoyalties) {
            for (uint256 i = 0; i < tokenIds.length; i++) {
                // get the royalty fee for the NFT
                (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);


                // transfer the royalty fee to the recipient if it's greater than 0
                if (royaltyFee > 0 && recipient != address(0)) {
                    if (baseToken != address(0)) {
                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);
                    } else {
                        recipient.safeTransferETH(royaltyFee);
                    }
                }
            }
        }
```

##### Instances (1)

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238-L249
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L271-L285

##### Recommendation

Combine the 2 operations into one iteration:

```Solidity
    for (uint256 i = 0; i < tokenIds.length; i++) {
        // transfer the NFT to the caller
        ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

        if (payRoyalties) {
            // get the royalty fee for the NFT
            (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

            // add the royalty fee to the total royalty fee amount
            royaltyFeeAmount += royaltyFee;

            // transfer the royalty fee to the recipient if it's greater than 0
            if (royaltyFee > 0 && recipient != address(0)) {
                if (baseToken != address(0)) {
                    ERC20(baseToken).safeTransfer(recipient, royaltyFee);
                } else {
                    recipient.safeTransferETH(royaltyFee);
                }
            }
        }
    }
```

#

### [R-06] `buyQuote` reverts with arithmetic underflow instead of meaningful message
##### Description

When buying and NFF, execution reaches `buyQuote` where, we have the following calculation:

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L701
```Solidity
    FixedPointMathLib.mulDivUp(outputAmount, virtualBaseTokenReserves, (virtualNftReserves - outputAmount));
```

`virtualNftReserves` is how many NFTs there are in the pool and `outputAmount` is how many are to be bought (multiplied by 1e18).

This operation underflows and reverts if there are less NFTs that intended to buy. The halting of the execution is normal, but a check should be done before hand to gracefully exit with a custom error/require message

##### Instances (1)

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L701

##### Recommendation

Add a require/check that there are enough NFTs to be bought before continuing the `buy` call.
#

## Non-critical Issues (1)
#


### [NC-01] misleading or incorrect documentation
##### Description

There are places in the project where there is either incorrect or misleading code documentation (either via NatSpec or inline).
These need to be corrected

##### Instances (2)

- `tokenURI` declares a `uri` return variable in NatSpec but it simply returns the value ([code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L160))
- `flashFee` declares a `feeAmount` return variable in NatSpec but it simply returns the value ([code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L749))

##### Recommendation

Resolve the mentioned issues
