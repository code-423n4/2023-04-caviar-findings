## Missing setter for `changeFee`
`setChangeFee()` is missing in PrivatePool. In the event an adjustment is needed due to incorrect initialization, there is no option for that. Consider implementing a setter for `changeFee` just like it has been done so for its all other counterparts.

## Sanity checks at the constructor
Adequate zero address and zero value checks should be implemented in [`initialize()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157-L200) of PrivatePool to avoid accidental error(s) that could result in non-functional calls associated with it particularly when assigning presumably immutable variables, i.e. `baseToken`, `nft`, and `changeFee`.


  