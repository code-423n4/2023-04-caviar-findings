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


# 3: ADD A LIMIT FOR THE MAXIMUM NUMBER OF CHARACTERS PER LINE

Vulnerability details

## Context:

The solidity documentation recommends a maximum of 120 characters.

For reference, see https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length

## Proof of Concept

> ***2 Files, 5 Instances*** 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L103

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L58-L60

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L63

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider adding a limit of 120 characters or less to prevent large lines.


# 4: GENERATE PERFECT CODE HEADERS EVERY TIME					

Vulnerability details

## Context:

We recommend using a header for Solidity code layout and readability

For reference, see https://github.com/transmissions11/headers

## Proof of Concept

All Contracts

 /*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/

### Tools Used

Manual Analysis


# 5: IMPORTS CAN BE GROUPED TOGETHER

Vulnerability details

## Context:

Imports can be grouped together.

## Proof of Concept

> ***File: EthRouter.sol*** 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L31-L38


> ***File: PrivatePool.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L27-L37

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider importing OZ first, then all interfaces, then all utils.
