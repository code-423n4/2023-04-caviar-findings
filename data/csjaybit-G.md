Ternary operation is cheaper than if-else statement
There are instances where a ternary operation can be used instead of if-else statement. In these cases, using ternary operation will save modest amounts of gas.
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L278-L282 
bolb above can be rewritten as
baseToken != address(0) ? ERC20(baseToken).safeTransfer(recipient, royaltyFee) : recipient.safeTransferETH(royaltyFee);
the above ternary operation will save around 500 gas per transaction,because of forge bug we cant see difference in gas after using ternary operation instead of if else option