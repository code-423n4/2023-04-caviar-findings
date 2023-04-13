## 1. Can set the owner in the state itself instead of calling the external function every time
The current implementation of PrivatePool retrieves the owner by calling the Factory.ownerOf(address(PrivatePool))
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L127-L132
```
    modifier onlyOwner() virtual {
        if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
            revert Unauthorized();
        }
        _;
    }
```
But it can be set in initialization itself in the state of the contract which saves the extra gas cost of calling an external contract.

## 2. Common memory variables created in each iteration can be set outside of the loop
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L329-L352
```
        for (uint256 i = 0; i < tokenIds.length; i++) {
            // transfer each nft from the caller
            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);


            if (payRoyalties) {
                // calculate the sale price (assume it's the same for each NFT even if weights differ)
                uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;


                // get the royalty fee for the NFT
                (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);


                // tally the royalty fee amount
                royaltyFeeAmount += royaltyFee;


                // transfer the royalty fee to the recipient if it's greater than 0
                if (royaltyFee > 0 && recipient != address(0)) {
                    if (baseToken != address(0)) {
                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);
                    } else {
                        recipient.safeTransferETH(royaltyFee);
                    }
                }
            }
        }
```
Here `salePrice` is common in each iteration but still initialized at each iteration.
It can be initialized outside the loop which saves considerable gas.

## 3. Unused arguments in `flashfee`
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L750-L752
```
    function flashFee(address, uint256) public view returns (uint256) {
        return changeFee;
    }
```
