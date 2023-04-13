## 1. Consider implementing 2-step ownership transfer for Factory
If the owner of the Factory accidentally transfers ownership to an incorrect address, protected functions may become permanently inaccessible.
```
    function transferOwnership(address newOwner) public virtual onlyOwner {
        owner = newOwner;

        emit OwnershipTransferred(msg.sender, newOwner);
    }
}
```

Suggestion: Handle ownership transfers with two steps and two transactions. First, allow the current owner to propose a new owner address. Second, allow the proposed owner (and only the proposed owner) to accept ownership, and update the contract owner internally.

## 2. There should be some upper limit on `protocolFeeRate` in Factory.sol

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141-L143
```
    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }
```
Otherwise, owner can set it to any arbitrary value.
## 3. Owner of PrivatePool should not be decided based on ERC721.ownerOf(address(privatePool))

Currently, the owner of the private pool is decided based on the holder of that NFTId, not the one who created it. Although it creates a market for PrivatePool nftId, it may lead to some undesired behavior.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L127-L132

```
    modifier onlyOwner() virtual {
        if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
            revert Unauthorized();
        }
        _;
    }
```
Assume the case where the creator of PrivatePool has accidentally transferred the nft to someone else. Now that person is the owner of the pool and can withdraw all liquidity the original creator had provided.

## 4. There should be some upper limit on changeFee and a function to change it

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L179

Currently, there is no upper limit on changeFee which can allow the owner to set it to any value.

Also, there is no way to change it once it's changed.

## 5. Wrong `tokenWeights` are emitted in the event when `merkleRoot = bytes(0)`
There is no check in `buy()`, `sell()` and `change()` function that the `tokenIds.length == tokenWeights.length`
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211-L289
```
emit Buy(tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);
```
This `tokenWeights` is emitted in the event. 
So, if by mistake any user submits a transaction with wrong `tokenWeights`, it will be emitted instead of the actual one

## 6. Most of the functions are prone to re-entrancy
- ERC721 has a receiver callback
- None of the important function of the protocol has a nonReentrant modifier
- Although the impact is not much, it's worth adding `nonReentrant` modifier to prevent reentrancy


## 7. Event missing `indexed` params
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L59-L68
```
    event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);     // @audit-issue - QA - event mising indexed
    event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
    event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);
    event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
    event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);
    event SetMerkleRoot(bytes32 merkleRoot);
    event SetFeeRate(uint16 feeRate);
    event SetUseStolenNftOracle(bool useStolenNftOracle);
    event SetPayRoyalties(bool payRoyalties);
```
Most of the above events are missing `indexed` arguments

