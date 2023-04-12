### PrivatePool.sol
Line: 259, Line 368
save gas address(factory) => factory


### Save Gas Without Deploying new contract and codes for PrivatePoolMetadata
convert PrivatePoolMetadata to library and can save lots of deploy fee and external contract call fee gas

### Unused import statement
At EthRouter.sol
remove ./interfaces/IStolenNftOracle.sol

### PrivatePool.sol
setAllParameters function can be changed to external






