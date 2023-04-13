### Same calculation twice
`netInputAmount - feeAmount - protocolFeeAmount` calculated twice, also it is checked arithmetic operation, so cheaper to store result in a stack variable than calculate it twice;
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L236
**Recommendation:**
We could store result to stack variable and use it instead.

### `public` functions to `external`
External call cost is less expensive than of public functions.
Contracts are allowed to override their parents’ functions and change the visibility from external to public.

There are multiple instances

### <x> += <y> costs more gas than <x> = <x> + <y> for state variables
Using the addition operator instead of plus-equals saves 113 gas.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L247
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L252
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L341
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L355

### Functions guaranteed to revert when called by normal users can be marked `payable``
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Use hardcode address instead address(this)
Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address.

References: https://twitter.com/transmissions11/status/1518507047943245824 - https://book.getfoundry.sh/reference/forge-std/compute-create-address