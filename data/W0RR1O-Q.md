## QA Report(low/non-critical)

[L-01] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256
================================================================================================================
In the function `buy()` and `sell()` of the contract `PrivatePool.sol` the function first set the variables `netInputAmount`,`feeAmount`,`protocolFeeAmount` and `weightSum` to be of type `uint256`. However, later on in the function the value of the variables are downcasted to `uint128` and is used to update the virtual reserves.

## Proof Of Concept
The `buy()` function in the contract `PrivatePool.sol`:
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-#L231

The `sell()` function in the contract `PrivatePool.sol`:
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301-#L324

