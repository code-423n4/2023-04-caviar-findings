## Gas Optimization Report

[G-01] `x += y` costs more gas than `x = x + y` for state variables(x-=y too)
===============================================================================
Using the addition operator instead of plus-equals saves `113 gas`. Subtractions act the same way.

There are nine instances found in the contract `PrivatePool.sol`:
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L247
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L252
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L341
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L355
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L678