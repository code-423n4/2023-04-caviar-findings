[N1]Events is missing indexed fields

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L41

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L58-L68

Index event fields make the field more quickly accessible to off-chain.Each event should use three indexed fields if there are three or more fields.

[N2]Use of bytes.concat() instead of abi.encodePacked()
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L19-L30

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L39-L50

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L59-L117

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)