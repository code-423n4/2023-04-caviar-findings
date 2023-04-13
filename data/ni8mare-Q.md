# Low Issues 

## [L-1] No zero address checks

The constructor in [EthRouter](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L90) and [PrivatePool](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L143) contracts does not check for zero address for the input parameters of type address.

In the Factory contract too, `setPrivatePoolMetadata` and `setPrivatePoolImplementation` should also check for zero address.

## [L-2] Return excess fees back to the caller of the flashloan

In the `PrivatePool` contract, the buy function returns any excess Eth sent for buying NFTs - 
`if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);`

But, the `flashloan` function does not return any excess Eth that might have been sent by the caller for paying the fee. Excess Eth should be sent back in order to remain fair to the caller. Add the following line at the end of the function- 

`if (baseToken == address(0)) msg.sender.safeTransferETH(msg.value - fee)`

# Non-Critical Issues

## [NC-1] Consider adding a check on the protocol fee 

The [setFeeRate](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L562) checks whether `newFeeRate` is too high - `if (newFeeRate > 5_000) revert FeeRateTooHigh()`.  
Consider adding a similar check for `protocolFeeRate` in the [setProtocolFeeRate](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141) function of the Factory contract. 

## [NC-2] Comments do not specify all the return values of a function in some cases.

For example, the comments above the [buy](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211) function does not specify all the return values. In this case `protocolFeeAmount` is not specified.

This is also true for `sell`, `change`, `buyQuote`, `sellQuote` functions.