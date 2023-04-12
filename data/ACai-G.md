# verifing `public` to `external` for saving gas

Suggest to verify the function visibility quantifiers from `public` to `external` for saving gas.

- Factory.setPrivatePoolMetadata, Line 129
- Factory.setPrivatePoolImplementation, Line 135
- Factory.setProtocolFeeRate, Line 141
- Factory.withdraw, Line 148

# verifing the parameter visibility to `immutable` for saving gas
PrivatePool.constructor, Line 143-147
These `factory`, `royaltyRegistry` and `stolenNftOracle` parameter only set in the constructor function. So suggest to add `immutable` for saving gas.
```solidity
    constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
        factory = payable(_factory);
        royaltyRegistry = _royaltyRegistry;
        stolenNftOracle = _stolenNftOracle;
    }
```