## QA Report(low/non-critical)

[L-01] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256
================================================================================================================
In the function `buy()` and `sell()` of the contract `PrivatePool.sol` the function first set the variables `netInputAmount`,`feeAmount`,`protocolFeeAmount` and `weightSum` to be of type `uint256`. However, later on in the function the value of the variables are downcasted to `uint128` and is used to update the virtual reserves.

## Proof Of Concept
The `buy()` function in the contract `PrivatePool.sol`:
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-#L231

The `sell()` function in the contract `PrivatePool.sol`:
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301-#L324

[L-02] Use of `block.timestamp` in the contract `EthRouter.sol`
================================================================
In the contract `EthRouter.sol` there are 4 instances where `block.timestamp` is used to to check whether or not a deadline has been passed. However, `block.timestamp` does not mean current time. Miners can influence the value of `blok.timestamp` to perform a Maximal Extractable Value (MEV) attack. Furthermore, miners can modify the timestamp by up to 900 seconds. 

## Proof Of Concept:
There are 4 instances in the contract `EthRouter.sol`:
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256
## Recommended Mitigation
Use `block.number` instead of `block.timestamp` or `now`. Furthermore, it is also recommended to use oracles.

[L-03] Update codes to avoid Compilation Errors
================================================
```
warning[2072]: Warning: Unused local variable.
   --> test/PrivatePool/Quotes.t.sol:115:37:
    |
115 |         (uint256 returnedFeeAmount, uint256 protocolFeeAmount) = privatePool.changeFeeQuote(inputAmount);
    |                                     ^^^^^^^^^^^^^^^^^^^^^^^^^
```
