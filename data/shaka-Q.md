# Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
|[L-01]| Ownership is transferred in one step | 1 |

## [L-01] Ownership is transferred in one step

The `Factory` contract uses Solmate's `Owned` abstract contract, which transfers the ownership in one step. This can lead to issues if `transferOwnership` is executed sending the wrong `newOwner` address, leaving the admin functions not accessible anymore.

### Recommended Mitigation Steps

Use OpeZeppelin's `Ownable2Step` contract or similar implementation.


# Non-Critical Issues

| |Issue|Instances|
|-|:-|:-:|
|[N-01]| Floating pragma  | 5 |
|[N-02]| Incorrect comments | 1 |
|[N-03]| No error message for revert | 1 |
|[N-04]| Internal function should start with underscore | 1 |
|[N-05]| Missing parameters in NatSpec | 2 |

## [N-01] Floating pragma

All contracts in scope (`Factory.sol`, `EthRouter.sol`, `PrivatePool.sol`, `PrivatePoolMetadata.sol` and `IStolenNftOracle.sol.sol`) use floating pragma for Solidity version.

```solidity
1   // SPDX-License-Identifier: MIT
2   pragma solidity ^0.8.19;
```

### Recommended Mitigation Steps

Lock Solidity version.

```diff
   // SPDX-License-Identifier: MIT
-  pragma solidity ^0.8.19;
+  pragma solidity 0.8.19;
```


## [N-02] Incorrect comments

The comment for `PrivatePool.sol:setFeeRate` is incorrect, as says this fee is used when changing NFTs, however for NFT changes `changeFee` is used instead.

```solidity
File: PrivatePool.sol
558    /// @notice Sets the fee rate. Can only be called by the owner of the pool. The fee rate is used to calculate the
559    /// fee amount when swapping or changing NFTs. The fee rate is in basis points (1/100th of a percent). For example,
560    /// 10_000 == 100%, 200 == 2%, 1 == 0.01%.
561    /// @param newFeeRate The new fee rate (in basis points)
562    function setFeeRate(uint16 newFeeRate) public onlyOwner {
```

### Recommended Mitigation Steps

```diff
    /// @notice Sets the fee rate. Can only be called by the owner of the pool. The fee rate is used to calculate the
-   /// fee amount when swapping or changing NFTs. The fee rate is in basis points (1/100th of a percent). For example,
+   /// fee amount when swapping. The fee rate is in basis points (1/100th of a percent). For example,
    /// 10_000 == 100%, 200 == 2%, 1 == 0.01%.
    /// @param newFeeRate The new fee rate (in basis points)
    function setFeeRate(uint16 newFeeRate) public onlyOwner {
```


## [N-03] No error message for revert

Reverting without an error message does not provide feedback to the user and hinders debugging.

```solidity
File: PrivatePool.sol
474     } else {
475         revert();
476     }
```

## [N-04] Internal function should start with underscore

```solidity
File: PrivatePoolMetadata.sol
112     function trait(string memory traitType, string memory value) internal pure returns (string memory) {
```

## [N-05] Missing parameters in NatSpec

`PrivatePool.sol:initialize()` NatSpec does not include `_changeFee` and `_payRoyalties` params.

```solidity
File: PrivatePool.sol
149     /// @notice Initializes the private pool and sets the initial parameters. Should only be called once by the factory.
150     /// @param _baseToken The address of the base token
151     /// @param _nft The address of the NFT
152     /// @param _virtualBaseTokenReserves The virtual base token reserves
153     /// @param _virtualNftReserves The virtual NFT reserves
154     /// @param _feeRate The fee rate (in basis points) 200 = 2%
155     /// @param _merkleRoot The merkle root
156     /// @param _useStolenNftOracle Whether or not the pool uses the stolen NFT oracle to check if an NFT is stolen
157     function initialize(
158         address _baseToken,
159         address _nft,
160         uint128 _virtualBaseTokenReserves,
161         uint128 _virtualNftReserves,
162         uint56 _changeFee,
163         uint16 _feeRate,
164         bytes32 _merkleRoot,
165         bool _useStolenNftOracle,
166         bool _payRoyalties
167     ) public {
```

`PrivatePool.sol:buyQuote()` NatSpec does not include return value `protocolFeeAmount`.

```solidity
File: PrivatePool.sol
689     /// @notice Returns the required input of buying a given amount of NFTs inclusive of the fee which is dependent on
690     /// the currently set fee rate.
691     /// @param outputAmount The amount of NFTs to buy multiplied by 1e18.
692     /// @return netInputAmount The required input amount of base tokens inclusive of the fee.
693     /// @return feeAmount The fee amount.
694     function buyQuote(uint256 outputAmount)
695         public
696         view
697         returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
```