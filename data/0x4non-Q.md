# QA

## Low

### L01 Unsafe downcasting

On [PrivatePool.sol#L230-L231](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L230-L231) there are two unsafe downcasting from `uint256` to `uint128`;
```solidity
        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
        virtualNftReserves -= uint128(weightSum);
```

I think is possible under certain cirscunstances that `weightSum > uint128.max` or `netInputAmount - feeAmount - protocolFeeAmount > uint128.max`

Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows/underflows when casting from other types

### L02 Its possible to reenter to `PrivatePool.change` function

Its factible to reenter to the `change` function if you send more ether than you supposed to, the line tha makes possible the reentrancy is on [PrivatePool.sol#L436](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L436)

I think its more practical to avoid native ETH in favour of always using WETH, this will make all contracts less complex.

If you want to test it out you could reuse [this test](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/test/EthRouter/Change.t.sol#L25-L57) and add a `receive() external payable` to reenter or do any action that you want before the protocol take you nft in give you and change it for another.


### L03 Missing `address(0)` checks

- `privatePoolMetadata` can be set to `address(0)` on [Factory.sol#L129-L131](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L131)
- `privatePoolImplementation` can be set to `address(0)` on [Factory.sol#L135-L137](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135-L137)


### L04 Missing bound check can set variables to extremes that make protocol impossible to use

- While `initializing` the `PrivatePool` there is no bound checks for `virtualBaseTokenReserves` or `virtualNftReserves`, [PrivatePool.sol#L177-L178](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L177-L178)
- On `setVirtualReserves` there is no bound checks for `virtualBaseTokenReserves` or `virtualNftReserves`, [PrivatePool.sol#L538-L545](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L545)
- `protocolFeeRate` doesnt have any bound check, allowing protocol to set it un a high values that could be interpretate as a rugpull, [Factory.sol#L141-L143](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141-L143)


## Non critical

### NC01 Missing event emission on critical functions

Consider add event emission on critical functions, especially when updating protocol parameters;
- `setPrivatePoolMetadata`, [Factory.sol#L129-L131](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L131)
- `setPrivatePoolImplementation`, [Factory.sol#L135-L137](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135-L137)
- `setProtocolFeeRate`, [Factory.sol#L141-L143](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141-L143)

### NC02 Avoid using magic numbers

On `PrivatePool` you see a number repeating all over `10_000` this is because its representing `BPS`, is recommend to set a constant for using it.

### NC03 Avoid empty revert message

Add a descriptive revert message (or better a custom error) on [PrivatePool.sol#L474](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L474)