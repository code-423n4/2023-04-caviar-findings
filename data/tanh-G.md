# `salePrice` should be calculated outside for loop

Link: https://github.com/code-423n4/2023-04-caviar/blob/463473a74640f2bf5907c0376959f1215e861fbc/src/PrivatePool.sol#L335

The `salePrice` value is calculated on every loop iteration even though it's always constant. It should be done once outside the for loop 