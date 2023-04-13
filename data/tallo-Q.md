[L-01] All cases of transferFrom and transfer should be replaced with their safe counterparts

To allow further interfacing with weirder ERC20's safeTransfer/safeTransferFrom are very useful. Especially when dealing with tokens that return ```false``` instead of reverting on failure theres a possibility of the protocol losing funds.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L365
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L527
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L115
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L152


[L-02] Mislead users can be tricked into depositing and losing their depeosited NFT's and tokens
The ```deposit``` inside ```PrivatePool.sol``` function has virtually no utility for anybody besides the private pool owner. Any funds/NFT's sent to this pool becomes owned by the pool owner with no chance of redeeming. Though extremely unlikely, a tricked/ignorant user can call this function and end up losing their deposited wealth. My suggestion is to add the onlyOwner access modifier to the function since the only individual who will use it legitimately is the owner.
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484-L507

