# Report
## Gas Optimizations ##
### [G-1]: Place subtractions where the operands can't underflow in unchecked {} block
**Context:**

1. ```if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);``` [L268](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L268) (msg.value - netInputAmount)
1. ```msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);``` [L436](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L436) (msg.value - feeAmount - protocolFeeAmount checked in L435)

**Description:**

Some gas can be saved by using an unchecked {} block if an underflow isn't possible because of a previous require() or if-statement.