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

[G-03] `Public` functions not called by the contract should be declared `external` instead
===========================================================================================
Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so.

There are four instances in the contract `EthRouter.sol`:
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L99
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L152
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L226
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254

There are two instances in the contract `Factory.sol`:
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L84
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148

There are six instances in the contract `PrivatePool.sol`:
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L212
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L306
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L393
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484
* https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514