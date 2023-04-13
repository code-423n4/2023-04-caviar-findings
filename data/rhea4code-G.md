# 1. Don't Initialize Variables with Default Value
### Description
If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

Uninitialized variables are assigned with the types default value.
Explicitly initializing a variable with it's default value costs unnecessary gas.

### Proof of Concept

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L106
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L116
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L134
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L159
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L161
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L183
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L239
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L261
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L265
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L284
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L119
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L106
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L237
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L272
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L328
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L446
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L496
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L518
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L673

# 2. Cache Array Length Outside of Loop
### Description
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Furthermore, there is no need to assign the initial value 0. This costs extra gas.

### Proof of Concept

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L106
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L115
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L116
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L134
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L159
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L161
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L182
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L183
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L239
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L261
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L265
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L284
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L119
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L236
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L272
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L446
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L467
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L518
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L668
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L672
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L673

# 3. Use != 0 instead of > 0 for Unsigned Integer Comparison
### Description
When dealing with unsigned integer types, comparisons with != 0 are cheaper than with > 0.

So, using > 0 costs more gas than != 0 when used on a uint in a require() statement.

### Proof of Concept

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L121
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L141
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L188
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L290
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L159
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L161
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L182
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L183
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L259
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L265
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L362
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L368
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L426
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L432
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L467
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489