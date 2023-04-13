## Table of Contents

- [L-01] Take special care of special NFTs that can be paused or evolved
- [L-02] Have an allowlist of tokens in order to prevent weird ERC20 tokens from bricking the protocol
- [N-01] Non-library/interface files should use fixed compiler versions, not floating ones
- [N-02] Don't use the latest version of solidity that has not been battle-tested yet

### [L-01] Take special care of special NFTs that can be paused or evolved

If an NFT can be paused like cryptokitties, then the privatePool will not be able to work as intended. Some NFTs can also be evolved, which will affect the pool price because the original NFT will be burnt and the new NFT will be replaced. 

in both crypto-kitty and in crypto-figher NFT, the transfer and transferFrom method can be paused.

In crypto-figher NFT:

https://etherscan.io/address/0x87d598064c736dd0C712D329aFCFAA0Ccc1921A1#code#L873

```
function transferFrom(
    address _from,
    address _to,
    uint256 _tokenId
)
    public
    whenNotPaused
{
```

In Crypto-kitty NFT:

https://etherscan.io/address/0x06012c8cf97BEaD5deAe237070F9587f8E7A266d#code#L615
```
function transferFrom(
    address _from,
    address _to,
    uint256 _tokenId
)
    external
    whenNotPaused
{
```

note the WhenNotPaused modifier.

Evolving NFTs, or dNFTs: https://chain.link/education-hub/what-is-dynamic-nft

Consider disallowing these types of NFT when creating PrivatePools.


### [L-02] Have an allowlist of tokens in order to prevent weird ERC20 tokens from bricking the protocol

There are some weird ERC20 tokens that have no decimal places, such as singularDTV 

https://etherscan.io/address/0xaec2e87e0a235266d9c5adc9deb4b2e29b54d009#readContract

If such a token is used in the NFT liquidity pairing pool, then a huge problem will arise when calling PrivatePool#changeFeeQuote() because the function attempts to subtract 4 from the baseToken of the decimals, which will result in revert due to underflow.

```
    function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
        // multiply the changeFee to get the fee per NFT (4 decimals of accuracy)
//@audit-- this line
        uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;
        uint256 feePerNft = changeFee * 10 ** exponent;


        feeAmount = inputAmount * feePerNft / 1e18;
        protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;
    }
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L731-L738

### [N-01] Non-library/interface files should use fixed compiler versions, not floating ones

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively. Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile locally.

```
- pragma solidity ^0.8.19;
+ pragma solidity 0.8.19;
```

### [N-02] Don't use the latest version of solidity that has not been battle-tested yet

The latest version of solidty may contain some unknown errors. Use a battle-tested version like 0.8.17 instead



