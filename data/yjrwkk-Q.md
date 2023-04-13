### [L-01] Lines too long

Keep line width to max 120 characters for better readability where possible.  

[src/PrivatePool.sol#L58](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L58)  
```
event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);
```
[src/PrivatePool.sol#L59](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L59)  
[src/PrivatePool.sol#L60](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L60)  
[src/PrivatePool.sol#L63](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L63)  
[src/PrivatePool.sol#L642](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L642)  
[src/PrivatePoolMetadata.sol#L47](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L47)  
[src/PrivatePoolMetadata.sol#L63](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L63)  
[src/PrivatePoolMetadata.sol#L103](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L103)  

### [L-02] Unspecific compiler version pragma

It is recommended to pin to a concrete compiler version.  

[src/EthRouter.sol#L2](https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L2)  
```
pragma solidity ^0.8.19;
```
[src/Factory.sol#L2](https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L2)  
[src/PrivatePool.sol#L2](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L2)  
[src/PrivatePoolMetadata.sol#L2](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L2)  
[src/interfaces/IStolenNftOracle.sol#L2](https://github.com/code-423n4/2023-04-caviar/tree/main/src/interfaces/IStolenNftOracle.sol#L2)  

### [L-03] Events associated with setter functions

Consider having events associated with setter functions emit both the new and old values instead of just the new value.  

[src/PrivatePool.sol#L544](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L544)  
```
emit SetVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);
```
[src/PrivatePool.sol#L555](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L555)  
[src/PrivatePool.sol#L570](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L570)  
[src/PrivatePool.sol#L581](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L581)  
[src/PrivatePool.sol#L592](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L592)  
