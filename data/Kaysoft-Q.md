## [LC-01] AVOID FLOATING PRAGMA
Lock the prama version to ensure the contracts are deployed with the same version that they were tested with.
Consider changing 
```
pragma solidity ^0.8.19;
```
to 
```
pragma solidity 0.8.19;
```
files: all files
see: https://swcregistry.io/docs/SWC-103

## [LC-02] USE THE SAFECAST LIBRARY FOR CASTING VALUES TO AVOID OVERFLOW/UNDERFLOW
File: https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230L231
```
virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
        virtualNftReserves -= uint128(weightSum);
```

## [NC-01] Lines too long
File: https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L58-L60
The Solidity style guide recommends maximum of 120 characters per line of code. Use a formatter to ensure code are formatted to follow this guidelines.

```
event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);
    event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
    event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
```

