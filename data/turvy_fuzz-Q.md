### Duplicated revert checks should be refactored to a modifier or function
This can be better handled and improve code readability

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256

### Lack of address(0) checks in the constructor
In case of incorrect address definition in the constructor , there is no way to fix it because of the variables are immutable.

**https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143**