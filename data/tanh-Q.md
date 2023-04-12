# ETH not returned to user if fee is overpaid on flash loan

https://github.com/code-423n4/2023-04-caviar/blob/463473a74640f2bf5907c0376959f1215e861fbc/src/PrivatePool.sol#L627

In the other functions of the `PrivatePool`, excess fee payments are returned to the user. This is inconsistent with the `flashLoan` function. In this case, if the `baseToken` is 0 (ETH), then the user does not receive any refund for paying too much in fee