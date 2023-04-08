## Non-Critical 

### [NC-01]

Simplification of the `try catch` block to make code more readable and the function more gas efficient.

```
// PrivatePool.sol
function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {
    // return if the NFT is owned by this contract
    try ERC721(token).ownerOf(tokenId) returns (address result) {
        return result == address(this);
    } catch {
        return false;
    }
}

// recommended change
function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {
    // return if the NFT is owned by this contract
    address owner = ERC721(token).ownerOf(tokenId);
    return owner == address(this);
}
```

### [NC-02]

If statement can be simplified into a ternary operator. Gas savings from require.

```
// Factory.sol
if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
    revert PrivatePool.InvalidEthAmount();
}

// recommended change
if (_baseToken == address(0) ? msg.value != baseTokenAmount : msg.value >= 0) {
    revert PrivatePool.InvalidEthAmount();
}

// Additional gas savings from merging into require
require((_baseToken == address(0) ? msg.value == baseTokenAmount : msg.value >= 0), PrivatePool.InvalidEthAmount());
```