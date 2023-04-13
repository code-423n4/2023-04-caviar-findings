## [GAS-01] x += y/x -= y costs more gas than x = x + y/x = x - y for state variables

Files: https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230L231

```
virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
        virtualNftReserves -= uint128(weightSum);
```

## [GAS-02] USE `assembly` to write address storage values

files: https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129-L131

```diff
function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
-       privatePoolMetadata = _privatePoolMetadata; //@audit centeralization. use timelock.
+ assembly{
+ sstore(privateMetadata.slot, _privatePoolMetadata);
+ }
    }

```

## [GAS-03] SET CONSTRUCTORS TO `payable` to save 13 gas each on deployment.
Setting the constructor payable removes the initial check opcodes of msg.value == 0 and saves 13 gas on deployment without security risk.

Files: https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L53
- https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L90
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143

