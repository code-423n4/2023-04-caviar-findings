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

[G-02] Functions guaranteed to revert when called by normal users can be marked `payable`
=========================================================================================
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided costs an average of about `53 gas` per call to the function, in addition to the extra deployment cost.

There are four instances found in the contract `Factory.sol`:
Gas saved per instance: `53 gas`
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148

There are six instances found in the contract `PrivatePool.sol`:
Gas saved per instance: `53 gas`
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587