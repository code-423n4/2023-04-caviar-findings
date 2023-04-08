https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#LL791C1-L792C1

function _getRoyalty() checks only if the royalty fee is greater than the sales price but what if it is equal then the owner of the pool gets nothing.


function _getRoyalty() also returns the royaltyfee reciever
I dont know the risk rating of the second but what if the royaltyfee reciever is a contract that does not recieve ethers then the whole pool won't work out since the buy and sell sends eth to the royaltyfee reciever