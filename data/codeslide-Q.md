### Low Risk Issues List

| Number | Issue | Instances |
| :----: | :---- | :-------: |
| [L-01] | Unspecific compiler version pragma | 5 |
| [L-02] | Unsafe casting of uints            | 4 |

#### [L-01] Unspecific compiler version pragma
The compiler version specified for a contract should be locked to the version it has been tested the most with. Locking the pragma helps ensure that contracts do not accidentally get deployed using, for example, the latest compiler which may have higher risks of undiscovered bugs. Contracts may also be deployed by others and the pragma indicates the compiler version intended by the original authors. [locking-pragmas](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

For example:

```solidity
// bad
pragma solidity ^0.8.9;

// good
pragma solidity 0.8.9;
```

```solidity
File: src/Factory.sol

2:    pragma solidity ^0.8.19;
```

```solidity
File: src/PrivatePoolMetadata.sol

2:    pragma solidity ^0.8.19;
```

```solidity
File: src/EthRouter.sol

2:    pragma solidity ^0.8.19;
```

```solidity
File: src/PrivatePool.sol

2:    pragma solidity ^0.8.19;
```

```solidity
File: interfaces/IStolenNftOracle.sol

2:    pragma solidity ^0.8.19;
```

#### [L-02] Unsafe casting of uints

Downcasting from uint256 in Solidity does not revert on overflow. This can easily result in undesired exploitation or bugs, since developers usually assume that overflows raise errors. [OpenZeppelin's SafeCast](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast) restores this intuition by reverting the transaction when such an operation overflows. Using this library instead of the unchecked operations eliminates an entire class of bugs, so it’s recommended to use it always.

For example:

```solidity
// Before
virtualNftReserves -= uint128(weightSum);
```

```solidity
// After
virtualNftReserves -= toUint128(weightSum);
```

```solidity
File: src/PrivatePool.sol

230:    virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
231:    virtualNftReserves -= uint128(weightSum);

323:    virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
324:    virtualNftReserves += uint128(weightSum);
```

### Non Critical Issues

| Number  | Issue | Instances |
| :-----: | :---- | :-------: |
| [NC-01] | Constants should be defined rather than using magic numbers| 6 |
| [NC-02] | Function grouping and ordering | 2 |
| [NC-03] | Order of layout | 3 |

#### [NC-01] Constants should be defined rather than using magic numbers

```solidity
File: src/PrivatePool.sol

172:   if (_feeRate > 5_000) revert FeeRateTooHigh();

471:   revert(add(32, returnData), returnData_size)

564:   if (newFeeRate > 5_000) revert FeeRateTooHigh();

703:   protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() / 10_000;
704:   inputAmount * feeRate / 10_000;

733:   uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;

737:   protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;
```

#### [NC-02] Function grouping and ordering

Functions should be grouped according to their visibility and ordered per the Solidity Style Guide. Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. See [Order of Functions](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-functions).

1. In [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol), the `receive()` function should come right after the constructor.
2. In [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol), the `receive()` function should come right after the constructor.

#### [NC-03] Order of layout

Elements in a contract should be ordered per the Solidity Style Guide. See [Order of Layout](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-layout).

1. In [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol), the errors defined on [L81-L84](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L81-L84) should be moved down after the state variable `royaltyRegistry` on [L86](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L86).
2. In [Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol), the events defined on [L41-L42](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L41-L42) should be moved after the state variables ending on [L51](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L51).
3. In [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol), the events and errors defined on [L58-L79](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L58-L79) should be moved after the state variables ending on [L125](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L125).