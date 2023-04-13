L1 : 

Add a constant or variable reasonable limit to protocol fee rate + fee rate. An user can create a pool with 50% fee rate which is unhealty from an economic point of view. 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#:~:text=///%20%40notice%20The%20buy,uint16%20public%20feeRate%3B 

Source of the 50% limit:
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#:~:text=if%20(_feeRate%20%3E%205_000)%20revert%20FeeRateTooHigh()%3B 
 

NC1  :

Lock pragmas to specific compiler version 

EthRouter.sol / Factory.sol / PrivatePool.sol / PrivatePoolMetadata.sol 

 

NC 2: 

Add a comment about the equivalence 10_000 /1 00% 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#:~:text=protocolFeeAmount%20%3D%20inputAmount%20*,feeRate%20/%2010_000%3B 

NC3 : 

Wrong comment. Change 1% to 10% .  
https://github.com/code-423n4/2023-04-caviar/blob/main/test/PrivatePool/Buy.t.sol#:~:text=factory.setProtocolFeeRate(1_000)%3B%20//%201%25 

 

NC4 : 

Two tests from Buy.t.sol failed when fee rate is set to a value different from 0. Modify the following tests to include fee rate management. 
Log : 

 Encountered 2 failing tests in test/PrivatePool/Buy.t.sol:BuyTest 

[FAIL. Reason: Assertion failed.] test_PaysRoyaltiesIfRoyaltyFeeIsSet() (gas: 373890) 

[FAIL. Reason: Assertion failed.] test_UpdatesVirtualReserves() (gas: 232177) 

 
NC5 : 

Add a comment to explain role of RoyalRegistry state and the need of ERC2791. 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#:~:text=address%20public%20immutable%20royaltyRegistry%3B 

 
NC6: 

Add a comment to mention the 5_000 limit to feerate. 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#:~:text=///%20%40notice%20The%20buy,uint16%20public%20feeRate%3B 

 

 