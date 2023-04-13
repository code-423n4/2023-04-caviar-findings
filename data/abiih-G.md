# Report

---

## Gas Optimizations

---

|     | Issue                                                                                                                                                                 |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | [Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead](#usage-of-uintsints-smaller-than-32-bytes-256-bits-incurs-overhead)                            |
| 2   | [Use external instead of public where possible](#use-external-instead-of-public-where-possible)                                                                       |
| 3   | [Functions Guaranteed To Revert When Called By Normal Users Can Be Marked Payable](#functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable) |
| 4   | [ Use calldata instead of memory for function arguments that do not get mutated](#use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated)      |

---

1. ### Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

   When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher.
   This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L51
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L88
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L91

```diff
-     uint16 public protocolFeeRate;
+     uint16 public protocolFeeRate;

```

2. ### Use external instead of public where possible
   Functions with the public visibility modifier are costlier than external. Default to using the external modifier until you need to expose it to other functions within your contract.

- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L602

```diff
-function initialize(
-        address _baseToken,
-       address _nft,
-      uint128 _virtualBaseTokenReserves,
-        uint128 _virtualNftReserves,
-        uint56 _changeFee,
-        uint16 _feeRate,
-        bytes32 _merkleRoot,
-        bool _useStolenNftOracle,
-        bool _payRoyalties
-    ) public {
+function initialize(
+        address _baseToken,
+        address _nft,
+        uint128 _virtualBaseTokenReserves,
+        uint128 _virtualNftReserves,
+        uint56 _changeFee,
+        uint16 _feeRate,
+        bytes32 _merkleRoot,
+        bool _useStolenNftOracle,
+        bool _payRoyalties
+    ) external {


```

3. ### Functions Guaranteed To Revert When Called By Normal Users Can Be Marked Payable

   If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
   The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587

4. ### Use calldata instead of memory for function arguments that do not get mutated
   Mark data types as calldata instead of memory where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as calldata. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies memory storage.
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385
   - https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L661
