# 1. Unsafe ERC20 Operation(s)

## Summary
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

To circumvent ```ERC20's``` approve functions race-condition vulnerability use OpenZeppelin's ```SafeERC20``` library's ```safe{Increase|Decrease}Allowance``` functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.

## Proof of Concept
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#115
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#152
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#365
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#527
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#651

## Tools Used
VS Code for manual review.

## Recommended Mitigation Steps
It's recommended to always either use OpenZeppelin's ```SafeERC20``` library or at least to wrap each operation in a require statement.

# 1. Unspecific Compiler Version Pragma

## Summary
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

## Proof of Concept
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L2
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L2
https://github.com/code-423n4/2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol#L2
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L2
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L2

## Tools Used
VS Code for manual review.

## Recommended Mitigation Steps
It is recommended to pin to a concrete compiler version.