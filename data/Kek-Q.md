# [L-01] PrivatePool.setAllParameters is callable by anyone

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L602-L609

All function calls in this function call have the `onlyOwner` modifier, it may make sense to also mark this function (setAllParameters) as `onlyOwner` as well. This may help prevent introducing future issues if this function is updated with the added bonus of simplifying the public/external facing APIs for this protocol.