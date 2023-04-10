# 1: FLOATING PRAGMA

Vulnerability details

## Context:

All the contracts in scope are floating the pragma version.


## Proof of Concept

All contracts in scope

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case of contracts in a library or a package.


# 2: PRAGMA VERSION ^0.8.19 VERSION TOO RECENT TO BE TRUSTED.

Vulnerability details

## Context:

Unexpected bugs can be reported in recent versions;
Risks related to recent releases
Risks of complex code generation changes
Risks of new language features
Risks of known bugs

0.8.19 (2023-02-22)
0.8.18 (2023-02-01)
0.8.17 (2022-09-08)
0.8.16 (2022-08-08)

For reference, see https://github.com/ethereum/solidity/blob/develop/Changelog.md  


## Proof of Concept

 > ***File: Factory.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L2


 > ***File: PrivatePoolMetadata.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L2


 > ***File: EthRouter.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L2


 > ***File: PrivatePool.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L2


> ***File: IStolenNftOracle.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/interfaces/IStolenNftOracle.sol#L2

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Use a non-legacy and more battle-tested version



