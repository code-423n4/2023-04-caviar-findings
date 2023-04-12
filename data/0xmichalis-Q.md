## Factory

1- Emit events in `setPrivatePoolMetadata`, `setPrivatePoolImplementation`, and `setProtocolFeeRate` so clients can monitor these admin functions easier by watching events instead of having to poll an RPC node. 
2- `setPrivatePoolMetadata` could furthermore implement [EIP-4906](https://eips.ethereum.org/EIPS/eip-4906) in case there are NFT marketplaces that cache metadata and keep track of changes via EIP-4906.

## PrivatePool

1- `_changeFee` and `_payRoyalties` are not documented in `initialize()`'s NatSpec
2- If a user sends more ETH than the necessary fee in `flashLoan()`, the excess ETH is not refunded back to the user as is the case with excess ETH provided in a `buy()` or a `change()`