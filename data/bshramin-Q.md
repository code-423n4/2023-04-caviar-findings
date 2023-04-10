# L1 - The pool takes more royalty fee than it spends
In sending the royalty fees the pool checks if the recipient address is zero and won't send it any tokens.
```solidity
(uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);
if (royaltyFee > 0 && recipient != address(0)) {
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L274


But when the pool is calculating the amount of royalty that should be payed by the user, it ignores the recipient address and always adds the amount to the royalty fee.
```solidity
(uint256 royaltyFee,) = _getRoyalty(tokenIds[i], salePrice);
royaltyFeeAmount += royaltyFee;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L244


This can cause taking more funds from the user than it should if the royalty recipient is set to address zero for some NFT.


# L2 - Royalty payment is unfair
User might end up paying more or less loyalty than he's supposed to. With the following code snippet we can see that the pool is calculating the royalty fee by dividing the total amount of tokens that the user is paying by the number of NFTs that he's buying.
```solidity
uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335

Imagine this scenario, 
|   |  Royalty Percentage  |  Price |
|---|---|---|
|  TOKEN1 |  10% | 10  |
|  TOKEN2 |  0% |  20 |
|  TOKEN3 |  0% |  30 |



Or on the other side, this scenario, 
|   |  Royalty Percentage  |  Price |
|---|---|---|
|  TOKEN1 |  10% | 30  |
|  TOKEN2 |  0% |  20 |
|  TOKEN3 |  0% |  10 |


In both these cases user ends up paying 2 tokens for royalties, but he should've only payed 1 token in the first case and 3 tokens in the second case.
This is not a fair calculation for royalty amounts.


# L3 - Flash loan multiple tokens at once 
I think it would be a good idea to make the flash loan function accept multiple tokens at once. This way we can save some gas and make the function more flexible.

So this function can be changed to accept a list of tokenIds instead of a single tokenId.
```solidity
function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data)
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L623


# L4 - flash loan might end up getting more fees than he's supposed to
In the `flashLoan` function, we have the check `msg.value < fee` so user might send more fees than he's supposed to and we are not sending the excess fees back to the user.
```solidity
if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635


# L5 - Change fee and flash fee should not have to be equal

In the private pool contract we are using the same variable for flashfee and change fee. Separating these two will provide the contract with more flexibility.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L751