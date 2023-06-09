## Missing setter for `changeFee`
`setChangeFee()` is missing in PrivatePool. In the event an adjustment is needed due to incorrect initialization, there is no option for that. Consider implementing a setter for `changeFee` just like it has been done so for its all other counterparts.

## Sanity checks at the constructor
Adequate zero address and zero value checks should be implemented in [`initialize()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157-L200) of PrivatePool to avoid accidental error(s) that could result in non-functional calls associated with it particularly when assigning presumably immutable variables, i.e. `baseToken`, `nft`, and `changeFee`.

## Typo mistakes
[File: Factory.sol#L166](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L166)

```diff
-    /// @param salt The salt that will used on deployment.
+    /// @param salt The salt that will be used on deployment.
```
## Unbounded loop in `deposit()`
The [for loops](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L239-L241) entailed could easily run out of gas since an NFT collection usually entails thousands of tokenIds. The impact is low since the user can always retry with a smaller array size after encountering a DoS. Where possible, comment with a suggested `tokenIds.length` so users get an idea what array size to work with or better yet set an upper bound to it in the function logic .    

## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

Consider fully equipping all contracts with complete set of NatSpec to better facilitate users/developers interacting with the protocol's smart contracts.

For instance, the function NatSpec of [`sell()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L291-L306) in PrivatePool will be perfect with missing @return protocolFeeAmount added.  

## Unspecific compiler version pragma
For some source-units the compiler version pragma is very unspecific, i.e. ^0.8.19. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

## Non-compliant contract layout with Solidity's Style Guide
According to Solidity's Style Guide below:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the `view` and `pure` functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Consider adhering to the above guidelines for all contract instances entailed.

## Code repeatedly used should be grouped into a modifier
Grouping similar/identical code block into a modifier makes the code base more structured while reducing the contract size.

Here are the four instances entailed:

[File: EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

```solidity
101:        if (block.timestamp > deadline && deadline != 0) {
102:            revert DeadlinePassed();
103:        }

154:        if (block.timestamp > deadline && deadline != 0) {
155:            revert DeadlinePassed();
156:        }

228:        if (block.timestamp > deadline && deadline != 0) {
229:            revert DeadlinePassed();
230:        }

256:        if (block.timestamp > deadline && deadline != 0) {
257:            revert DeadlinePassed();
258:        }
```
## Gas griefing/theft is possible on unsafe external call
`return` data (bool success,) has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0). Thus, this storage disappears and may come from external contracts a possible gas grieving/theft problem is avoided as denoted in the link below:

https://twitter.com/pashovkrum/status/1607024043718316032?t=xs30iD6ORWtE2bTTYsCFIQ&s=19

Here is a specific instance entailed:

[File: PrivatePool.sol#L461](PrivatePool.sol#L461](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L461)

```solidity
        (bool success, bytes memory returnData) = target.call{value: msg.value}(data);
```
## Events associated with setter functions
Consider having events associated with setter functions emit both the new and old values instead of just the new value.

Here are some of the instances entailed:

[File: PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

```solidity
544:        emit SetVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);

555:        emit SetMerkleRoot(newMerkleRoot);

570:        emit SetFeeRate(newFeeRate);

581:        emit SetUseStolenNftOracle(newUseStolenNftOracle);

592:        emit SetPayRoyalties(newPayRoyalties);
```
## `_getRoyalty()` is not a public view function
Unlike [`getRoyalty()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L301-L316) in EthRouter, `_getRoyalty()` in PrivatePool has no public visibility. Hence, users could not use it to retrieve info on `royaltyFee` to help them make good decisions whether or not to deal with certain tokenIds that might charge higher than expected fees.

Consider making it public where possible:

[File: PrivatePool.sol#L778-L793](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L778-L793) 

```diff
    function _getRoyalty(uint256 tokenId, uint256 salePrice)
-        internal
+        public
        view
        returns (uint256 royaltyFee, address recipient)
    {
        // get the royalty lookup address
        address lookupAddress = IRoyaltyRegistry(royaltyRegistry).getRoyaltyLookupAddress(nft);

        if (IERC2981(lookupAddress).supportsInterface(type(IERC2981).interfaceId)) {
            // get the royalty fee from the registry
            (recipient, royaltyFee) = IERC2981(lookupAddress).royaltyInfo(tokenId, salePrice);

            // revert if the royalty fee is greater than the sale price
            if (royaltyFee > salePrice) revert InvalidRoyaltyFee();
        }
    }
```
## Timelock for setter functions
Changes made via the setter functions in PrivatePool are sensitive transactions that may go against users` will. As such, users of the system should have assurances about the behavior of the changed action(s) they’re about to take.

For instance, a user's [buying transaction](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211) could be preceded or frontrun inadvertently by [`setFeeRate()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562-L571) leading to the user paying more [`feeAmount`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L222) than originally intended.

Consider implementing a time lock by making the crucial changes require two steps with a mandatory time window between them. The first step merely broadcasts to users that a particular change is coming, and the second step commits that change after a suitable waiting period. This allows users that do not accept the change to refrain from calling fee charging functions nearing the changing time. 

## Unrestricted `deposit()`
`deposit()` in both [EthRouter](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L219) and [PrivatePool](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484) is a public unrestricted payable functions. Ample measures will need to be in place to avoid purely non-owners, i.e users unrelated to the owner, from calling this sensitive function considering any funds and NFTs deposited into PrivatePool are going to be irretrievable unless the owner is kind enough withdrawing on their behalf and then transferring those assets back to the careless users. That said, the system should look into a better measure such as having only role specific users assigned by the owner to meddle with this function.    

## Uninitialized `cooldownPeriod` in StolenNftFilterOracle
[`cooldownPeriod`](https://github.com/outdoteth/caviar/blob/main/src/StolenNftFilterOracle.sol#L12) in StolenNftFilterOracle is currently in its default value, i.e. `0`. If it were to be set to a non-zero threshold, `validateTokensAreNotStolen()` externally called by [`sell()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L316-L318) and [`change()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L400-L402) from PrivatePool might more frequently encounter function revert due to the restricting require statement on line 67 below:

[File: StolenNftFilterOracle.sol#L48-L69](https://github.com/outdoteth/caviar/blob/main/src/StolenNftFilterOracle.sol#L48-L69) 

```solidity
    function validateTokensAreNotStolen(address tokenAddress, uint256[] calldata tokenIds, Message[] calldata messages)
        public
        view
    {
        if (isDisabled[tokenAddress]) return;

        for (uint256 i = 0; i < tokenIds.length; i++) {
            Message calldata message = messages[i];

            // check that the signer is correct and message id matches token id + token address
            bytes32 expectedMessageId = keccak256(abi.encode(TOKEN_TYPE_HASH, tokenAddress, tokenIds[i]));
            require(_verifyMessage(expectedMessageId, validFor, message), "Message has invalid signature");

            (bool isFlagged, uint256 lastTransferTime) = abi.decode(message.payload, (bool, uint256));

            // check that the NFT is not stolen
            require(!isFlagged, "NFT is flagged as suspicious");

            // check that the NFT was not transferred too recently
67:            require(lastTransferTime + cooldownPeriod < block.timestamp, "NFT was transferred too recently");
        }
    }
``` 
## `_verifyMessage()` does not check if `ecrecover` return value is 0
`_verifyMessage()` of ReservoirOracle calls the Solidity `ecrecover` function directly to verify the given signatures.

The return value of ecrecover may be 0, which means the signature is invalid, but the check can be bypassed when signer is 0.

Consequently, stolen NFTs could end up dodging the checks and end up being sold/changed into PrivatePool.

[File: ReservoirOracle.sol#L86-L109](https://github.com/reservoirprotocol/oracle/blob/main/contracts/ReservoirOracle.sol#L86-L109)

```solidity
        address signerAddress = ecrecover(
            keccak256(
                abi.encodePacked(
                    "\x19Ethereum Signed Message:\n32",
                    // EIP-712 structured-data hash
                    keccak256(
                        abi.encode(
                            keccak256(
                                "Message(bytes32 id,bytes payload,uint256 timestamp)"
                            ),
                            message.id,
                            keccak256(message.payload),
                            message.timestamp
                        )
                    )
                )
            ),
            v,
            r,
            s
        );

        // Ensure the signer matches the designated oracle address
        return signerAddress == RESERVOIR_ORACLE_ADDRESS;
```
Use the recover function from OpenZeppelin's ECDSA library for signature verification. Additionally, perform [a zero address check on `reservoirOracleAddress` at the constructor](https://github.com/reservoirprotocol/oracle/blob/main/contracts/ReservoirOracle.sol#L27-L29) where possible. 