Low issue 1: Reentrancy guard is highly recommended. Protocol doesn't specify usage of ERC777 tokens as base tokens.

 There are several places in the code that could trigger callbacks(NFT safe transfers, ETH/ERC20 safe transfers, onFlshLoan callback etc.). Although it appears that there are no references to  storage state after these calls, it's hard to rule out possible reentracy attacks as most of the safe transfers are happening inside the loop. To avoid hidden surprises, it's highly recommended apply Reentrancy guard.

Also protocol doesn't specify whether ERC777 tokens can be used as base tokens or not. To avoid possible risks, use Reentrancy guard.

Mitigation:
Add openzeppelin nonReentrant modifier to following functions.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211-L214
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L301-L306
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L623-L626



Low issue 2: Max limit for royalty fee percentage is not set.

Most of the market places set max limit for royalty fee percentage and revert the transaction if royalty fee exceeds max limit.

