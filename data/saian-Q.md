
# Low Severity findings

# EthRouter does not support ERC20 tokens

EthRouter contract does not support ERC20 base tokens, so using the contract to interact with private pool with ERC20 base token will give unexpected results

Function `PrivatePool#buy` will revert as tokens are transferred from `msg.sender` i.e.,EthRouter  

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L256

```solidity
    ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);
```

Function `EthRouter#sell` does not transfer the assets back to the user, so the ERC20 tokens will be stuck in the EthRouter contract

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L365

```solidity
    ERC20(baseToken).transfer(msg.sender, netOutputAmount);
```

## Attacker can front-run `create` to steal tokens sent to predetermined address

predictPoolDeployment function is used to determine address of the pool before deployment. So users can transfer tokens to the predetermined address before deployment. 
But an attacker can front run the user and create a pool using the same `salt` value and withdraw all the tokens in the address.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L168

```solidity
    function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {     
        predictedAddress = privatePoolImplementation.predictDeterministicAddress(salt, address(this)); 
    }
```

## If `royaltyFee > 0` and `recipient == address(0)` users will be overcharged

In functions `buy` and `sell` royalty fees are calculated from the `_getRoyalty` function, but the fees are sent only if the recipient is not address(0). So if royalty fees is greater than 0 but recipient is address(0), users will be overcharged

Recipient address is not validated during calculaing royalty fees

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L242

```solidity
    if (payRoyalties) {
        // get the royalty fee for the NFT
        (uint256 royaltyFee,) = _getRoyalty(tokenIds[i], salePrice);    

        // add the royalty fee to the total royalty fee amount
        royaltyFeeAmount += royaltyFee;
    }

    // add the royalty fee amount to the net input aount
    netInputAmount += royaltyFeeAmount;
```

Fees is sent to the recipient only if not address(0)

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L277

```solidity
    if (royaltyFee > 0 && recipient != address(0)) {    
        if (baseToken != address(0)) {
            ERC20(baseToken).safeTransfer(recipient, royaltyFee);
        } else 
            recipient.safeTransferETH(royaltyFee);  
        }
    }
```

## In function `buy` and `sell` if one royalty recipients reverts whole transaction will fail

In function `buy` and `sell`, royalty fees are sent to the recipients for each NFT token. If one recipient reverts the whole function will fail

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L281

```solidity
    if (royaltyFee > 0 && recipient != address(0)) {    
        if (baseToken != address(0)) {
            ERC20(baseToken).safeTransfer(recipient, royaltyFee);
        } else {
            recipient.safeTransferETH(royaltyFee);  
        }
    }
```

## Virtual reserves are not updated in `change`, `deposit` and `withdraw`

In function `change` if the input token weight is greater than the output tokens weights, the difference `inputWeightSum - outputWeightSum` is not added to the reserve value.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L413

```solidity
    if (inputWeightSum < outputWeightSum) revert InsufficientInputWeight();
```

During deposit nft token is transferred into the contract, but the reserves not not updated

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L497

```solidity
    ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
```

During withdraw if the `_nft` is equal to the pool nft, pool reserves are not decreased.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L519

```solidity
    ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
```

## changeFeeQuote reverts for tokens with decimals less than 4

In function `changeFeeQuote` if base token decimals is less than 4, function will revert due to underflow.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L733

```solidity
    uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;
```

## In `changeFeeQuote` if `changeFee == 0` protocolFeeAmount becomes 0

In `changeFeeQuote` protocolFeeAmount is calculated from `feeAmount`. So if pool owner sets changeFee to 0 during initialization, protocol fees becomes 0

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L737

```solidity
    uint256 feePerNft = changeFee * 10 ** exponent;

    feeAmount = inputAmount * feePerNft / 1e18;
    protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000; 
```

## Limit native fund transfer

Since contracts expect ether transfer from a single source, it is best to restrict ether transfer to prevent accident transfer of ether.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L134

```solidity
    receive() external payable {}
```

# Non-critical findings

## Unused import statement

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L5

```soliditys
    import {Base64} from "openzeppelin/utils/Base64.sol";
```

## Wrong data type used in ERC721.ownerOf() function

ERC721#ownerOf function expects integer of type uint256, but in modifier onlyOwner an integer of type uint160 is used

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L128

```solidity
    modifier onlyOwner() virtual {
        if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {  
            revert Unauthorized();
        }
        _;
    }
```

## Events not emitted for critical variable change

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129

```solidity
    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
        privatePoolMetadata = _privatePoolMetadata;
    }
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135

```solidity
    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141

```solidity
    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }
```

## Array lengths are not validated

In functions `buy`, `sell` and `change`, tokens and weight array lengths are not validated to be equal. If the arrays lengths are different, it may lead to unexpected results.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L301

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L385


If weights array is less than token ids length the function will fail silently

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L678

```
    for (uint256 i = 0; i < tokenIds.length; i++) {
        // create the leaf for the merkle proof
        leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

        // sum each token weight
        sum += tokenWeights[i]; 
    }
```
## Unused function params

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L750

```solidity
    function flashFee(address, uint256) public view returns (uint256) {
        return changeFee;
    }
```