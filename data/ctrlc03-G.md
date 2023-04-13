# Constructor could be made payable to save gas on deployment 

The constructor for `PrivatePool.sol` could be made payable in order to save gas on deployment. In more details, one can cut out 10 opcodes in the creation-time EVM bytecode by adding the payable modifier. 

Code location: https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143

# Save gas by removing Ether refund

Users are refunded any excess Ether in the `buy` and `change` functions of `PrivatePool.sol`. However, it is more gas efficient to do a check that the `msg.value` is equal to the expected amount (`netInputAmount` for `buy` and `feeAmount + protocolFeeAmount` for `change`) and just revert if not. This way an user could only call the function with the correct amount and save on unneded  transfers. 

`buy` function:

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L268

`change` function:

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L435-L436