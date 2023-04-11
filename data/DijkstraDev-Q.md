#### [L-01] Lack of safety checks in `create()` function params.
[Factory.sol#L71](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L71)

#### [QA-01] Simplification posibility
`tokenIds` is looped through twice, first to add the fees paid to the total, and then to send the fees to the `recipient`. It can be simplified as it is in `sell()`.
[PrivatePool.sol#L271](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L271)

#### [QA-03] Typo
[EthRouter.sol#L169](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L169)
