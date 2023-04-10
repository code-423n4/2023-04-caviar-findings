# GR1 - Fail fast(I)
In the eth router contract we should change this check:
```solidity
if (block.timestamp > deadline && deadline != 0) {
    revert DeadlinePassed();
}
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101

to this:
```solidity
if (deadline != 0 && block.timestamp > deadline) {
    revert DeadlinePassed();
}
```

With this change we will consume the same amount of gas if the deadline is set but we will consume less gas if the deadline is equal to zero.


# GR3 - Fail fast(II)
In the `buy` function of the private pool contract, we should move this check to the beginning of the function.
```solidity
if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225


# GR2 - Fail fast(III)
There is a check in the `Initialize` function:
```solidity
if (_feeRate > 5_000) revert FeeRateTooHigh();
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L172

This can be moved to the beginning of the `create` function in the `Factory` contract.
This way we will prevent spending a lot of gas for cloning the contract and calling initialize when the check fails.

