## Summary
### Low Risk Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]| Reorg attack possibility in pool factory  | 1 |
|[L-02]| Missing events on sensetive changes  | 3 |
|[L-03]| Pool and protocol owners could accidentally/intentionally front-run users with increasing fee | 2 |
|[L-04]| Pool owners could accidentally/intentionally front-run users with changing reserves | 1 |
|[L-05]| decimals() function is not guaranteed in ERC20 | 2 |
|[L-06]| `changeFee` parameter can not be updated after initialization | 1 |
|[L-07]| Malicious collection owner could steal all base tokens by updating royalty during calls | 1 |
|[L-08]| `flashLoan()` function allows user to transfer arbitrary tokens from pool address | 1 |
|[L-09]| Missing array length comparison | 1 |
|[L-10]| Use safeTransferOwnership instead of transferOwnership function | 1 |

### Non-Critical Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
| [N-01]|Misleading comment|1|


* * *
#### [L-01] Reorg attack possibility in pool factory 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L92
```solidity
File: Factory.sol
92:         privatePool = PrivatePool(payable(privatePoolImplementation.cloneDeterministic(_salt))); 
```
The factory contract uses the salt parameter during the cloning process when creating a new pool. However, this salt value is user-inputted, which could lead to a potential attack if the chain experiences a reorg incident.
In this scenario, an attacker could redeploy a pool with the same address, and any user who approved assets for the original pool could potentially have their assets stolen.
**Reccomendation:** Consider mixin `msg.sender` address into `salt`.

#### [L-02] Missing events on sensetive changes
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129-L143
```solidity
File: Factory.sol
129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner { // @audit-ok NC: missing events 
130:         privatePoolMetadata = _privatePoolMetadata;
131:     }
132: 
133:     /// @notice Sets the private pool implementation contract that newly deployed proxies point to.
134:     /// @param _privatePoolImplementation The private pool implementation contract.
135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
136:         privatePoolImplementation = _privatePoolImplementation;
137:     }
138: 
139:     /// @notice Sets the protocol fee that is taken on each buy/sell/change. It's in basis points: 350 = 3.5%.
140:     /// @param _protocolFeeRate The protocol fee.
141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner { // @audit-ok front-run fee increase + no upper bound
142:         protocolFeeRate = _protocolFeeRate;
143:     }
```
Missing emitting events during change of sensitive protocol parameters. **Recommendation:** Consider adding appropriate events.

#### [L-03] Pool and protocol owners could accidentally/intentionally front-run users with increasing fee
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562
```solidity
File: Factory.sol
141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner { 
142:         protocolFeeRate = _protocolFeeRate;
143:     }
File: PrivatePool.sol
562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {
563:         // check that the fee rate is less than 50%
564:         if (newFeeRate > 5_000) revert FeeRateTooHigh();
565: 
566:         // set the fee rate
567:         feeRate = newFeeRate;  
568: 
569:         // emit the set fee rate event
570:         emit SetFeeRate(newFeeRate);
571:     }
```
While a user expects to pay a specific fee size for a trade, the protocol or pool owner could potentially front-run the user's trade with a fee-increasing call, resulting in the user receiving a lower amount of funds than expected.
**Recommendation:** Consider adding a timelock of fee change and upper-bound for max fee size.

#### [L-04] Pool owners could accidentally/intentionally front-run users with changing reserves
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538
```solidity
File: PrivatePool.sol
538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner { 
539:         // set the virtual base token reserves and virtual nft reserves
540:         virtualBaseTokenReserves = newVirtualBaseTokenReserves;
541:         virtualNftReserves = newVirtualNftReserves;
542: 
543:         // emit the set virtual reserves event
544:         emit SetVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);
545:     }
```
While a user expects to pay a specific reserve values (=price), the pool owner could potentially front-run the user's trade with areserves changing call, resulting in the user receiving a lower amount of funds than expected.
**Recommendation:** Consider adding a timelock of reserves change.

#### [L-05] `decimals()` function is not guaranteed in ERC20
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L731-L746
```solidity
File: PrivatePool.sol
731:     function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) { 
732:         // multiply the changeFee to get the fee per NFT (4 decimals of accuracy)
733:         uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;
734:         uint256 feePerNft = changeFee * 10 ** exponent;
735: 
736:         feeAmount = inputAmount * feePerNft / 1e18;
737:         protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;
738:     }
...
742:     function price() public view returns (uint256) { 
743:         // ensure that the exponent is always to 18 decimals of accuracy
744:         uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals()); 
745:         return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;
746:     }

```
According to the [EIP-20](https://eips.ethereum.org/EIPS/eip-20) standard, the `decimals()` function is considered optional for ERC-20 tokens, meaning that tokens without this function may be unable to participate in certain protocol functionalities, such as the `change()` function.
**Recommendation:** Consider updating functions that calls `decimals()` function.

#### [L-06] `changeFee` parameter can not be updated after initialization
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L179
```solidity
File: PrivatePool.sol
179:         changeFee = _changeFee;
```
The `changeFee` parameter is set during initialization but cannot be updated after that. This is inconsistent with other fees such as `protocolFeeRate` and `feeRate`, which can be updated at any time.
**Recommendation:** Consider adding a function that allows the `changeFee` to be updated.

#### [L-07] Malicious collection owner could steal all base tokens by updating royalty during calls
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L274
```solidity
File: PrivatePool.sol
244:         (uint256 royaltyFee,) = _getRoyalty(tokenIds[i], salePrice); 
246:         // add the royalty fee to the total royalty fee amount
247:         royaltyFeeAmount += royaltyFee;
...
251:         // add the royalty fee amount to the net input aount
252:         netInputAmount += royaltyFeeAmount;
...
273:         // get the royalty fee for the NFT
274:         (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice); 
...
281:         recipient.safeTransferETH(royaltyFee);
```
The `buy()` function in `PrivatePool.so`l calls the `royaltyInfo()`(through `_getRoyalty()`) function on NFT collection twice during execution - once to account for the total amount of royalties that the trade pays, and a second time to send the royalties themselves.
However, a malicious collection owner could construct the royaltyInfo() function in a way that returns a bigger amount of royalties during the second call, leading to a drain of the pool's assets.

**Recommendation:** Consider storing the royalty info during the first call to `royaltyInfo()` and sending the stored amounts instead of second call. This approach would also likely save some gas.

#### [L-08] `flashLoan()` function allows user to transfer arbitrary tokens from pool address
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L623
```solidity
File: PrivatePool.sol
623:     function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data)
624:         external
625:         payable
626:         returns (bool)
627:     {
...
637:         // transfer the NFT to the borrower
638:         ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);
...
647:         // transfer the NFT from the borrower
648:         ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId); 
```
The `flashLoan()` function allows users to input an arbitrary token address, which could lead to the contract executing a transfer of arbitrary tokens previously transferred to the pool address. This could potentially lead to the pool address being blacklisted if the transferred token is somehow "banned", as it would appear to an external viewer that the pool address intentionally transferred the banned token.

**Recommendation:** Consider setting the NFT address for flash loans only to the pool's base NFT.

#### [L-09] Missing array length comparison
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L673
```solidity
File: PrivatePool.sol
673:         for (uint256 i = 0; i < tokenIds.length; i++) { 
674:             // create the leaf for the merkle proof
675:             leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
676: 
677:             // sum each token weight
678:             sum += tokenWeights[i];
679:         }

```
Missing array length comparison could lead to unexpected behavior.
**Recommendation:** Consider adding an equal array length check.

#### [L-10] Use safeTransferOwnership instead of transferOwnership function
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L37
```solidity
File: Factory.sol
37: contract Factory is ERC721, Owned {
```
**Recommendation:** Consider using more safe 2 step transfer onwership OZ [contract](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol).

#### [N-01] Misleading comment
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L375-L377
```solidity
File: PrivatePool.sol
375:     /// @notice Changes a set of NFTs that the caller owns for another set of NFTs in the pool. The caller must approve
376:     /// the pool to transfer the NFTs. The sum of the caller's NFT weights must be less than or equal to the sum of the
377:     /// output pool NFTs weights. The caller must also pay a fee depending the net input weight and change fee amount.
```
Comment says that *The sum of the caller's NFT weights must be less than or equal to the sum of the output pool NFTs weights* while code and protocol logic stay on a requirement that input weights sum should be >= output.