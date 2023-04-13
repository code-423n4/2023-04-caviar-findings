## Use the latest solidity version with a stable pragma statement
[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L2), [PrivatePoolMetadata.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L2), [IStolenNftOracle.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol#L2), [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L2), [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L2)

## Missing zero address checks
Factory.sol - [130](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129-L131), [135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135-L137),
EthRouter.sol - [91](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90-L92)
PrivatePool.sol - [143](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143-L147), [157](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157-L183)

## Missing arrays length check
PrivatePool.sol - [211](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211), [661](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L661-L664)

## Unsafe downcasting
PrivatePool.sol - [230](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230), [231](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231) 

## Use constants instead of magic numbers
PrivatePool.sol - [668](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L668), [703](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L703), [704](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L704), [721](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L721), [722](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L722),  [736](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L736), [737](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L737)

## `execute()` is missing nonreentrant modifier
PrivatePool.sol - [459](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459)

## Missing revert message
PrivatePool.sol - [474](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L474)

## Missing event for important parameter updates
Factory.sol - [129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129-L131),  [135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135-L137), [141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141-L143)