--- PrivatePool.sol ---

A. Fee structure is inconsistent for buys/changes. In `buy()` and `sell()` function protocolFee is based on a percentage of the input/output amount but in `change()` the fee is based on the pool fee not the input amount. It should be based on the input/output amount for consistency

B. Several places use the `ERC20.decimals()` function. `decimals()` is not a required part of the ERC20 standard, and not every token uses it. If the baseToken does not support decimals then the pool will be unusable since baseToken cannot be changed once initialized. Proper procedure is to wrap the decimals call in a "try-catch" block where decimals is default set at 18 if the call fails.
Lines 733 & 744, 
```
try ERC20(baseToken).decimals() returns (uint8 decimals) {
     exponent = 36 - decimals;
   } catch {
     // handle the exception here, e.g. set a default value
     exponent = 18;
```
https://eips.ethereum.org/EIPS/eip-20

C. Functions that take in multiple arrays as parameters do not verify that the arrays are of equal length. This can result in unexpected outcomes and transaction failures.

D. `FlashFee` is a flat amount, meaning the cost to flash loan an NFT at floor price is the same as one worth 5x the floor price. `FlashFee` should instead be based on the weigh of the NFT being flashloaned, with more expensive NFTs charging a higher fee.

E. All external facing functions should have `nonReentrant` tagged to prevent reentrancy vulnerabilities.