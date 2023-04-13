## L-01 Solidity version 0.8.19 is too recent and cannot be trusted


## L-02 Attribute is not updated when fees change 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L43-L44

The owner has the ability to change these values at will but the function attribute is not updated to reflect it 

