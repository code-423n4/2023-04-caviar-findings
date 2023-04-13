[L-01] All cases of transferFrom and transfer should be replaced with their safe counterparts

To allow further interfacing with weirder ERC20's safeTransfer/safeTransferFrom are very useful. Especially when dealing with tokens that return ```false``` instead of reverting on failure.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L365
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L527
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L115
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L152