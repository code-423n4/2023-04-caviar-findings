changeFeeQuote assumes all base tokens will be of decimals of at > 4 decimals. However this would lead to issues whereby baseToken is of 2 decimal place.

code: https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L733-L734