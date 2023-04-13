### [Low-01] While setting ```virtualNftReserves``` Contract doesn't checking Its scaled up or not
According to Code comments
```/// @dev The virtual NFT reserves that a user sets. If it's desired to set the reserves to match 16 NFTs then the
    /// virtual reserves should be set to 16e18. If weights are enabled by setting the merkle root to be non-zero then
    /// the virtual reserves should be set to the sum of the weights of the NFTs; where floor NFTs all have a weight of
    /// 1e18. A rarer NFT may have a weight of 2.3e18 if it's 2.3x more valuable than a floor.
```
Should scaled up by 10e18, 
may be creation of pool user may input wrong value
*Instances()*
```
File: 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L178
```
### [Low-02] Absence Of upper bound for ```changeFee``` During contract initialization and in further fee change functions.
Some upperbound should remain on chageFee parameter. Otherwise Pool owner could front-run and set fee to arbitary amount and draw more fund from user than intended.
*Instances()*
```
File: 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L179
```

### [Low-03] Functions are expose to re-entrancy attacks
```buy()```, ```sell()```, ```change()```
Now there is no significant affect by reentrancy in pool, but it may affect future protocols that use this protocol as base.
Should use Openzeppelin Reentrancy gaurd when functions making external calls.


```
File: 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L280
```
### [Low-04] Nft transfer will be failed as is absence of approval
Function directly trying to send Nft Without approval, 
First pool have to approve user to transfer Nft on behalf of it.
```solidity 
          for (uint256 i = 0; i < tokenIds.length; i++) {
            // transfer the NFT to the caller
            ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]); // @audit i think missing approval
```

```
File: 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238-L240
```
### [Low-05] Precision loss due to Division after Multiplication
```solidity
uint256 inputAmount =
            FixedPointMathLib.mulDivUp(outputAmount, virtualBaseTokenReserves, (virtualNftReserves - outputAmount));

        protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() / 10_000; // @audit precision loss
        feeAmount = inputAmount * feeRate / 10_000; // @audit precision loss
        netInputAmount = inputAmount + feeAmount + protocolFeeAmount;
```
*Instances(2)*
```
File: 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L700-L706
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L719-L723

```
