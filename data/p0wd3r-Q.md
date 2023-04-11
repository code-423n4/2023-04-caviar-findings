# Replace `transferFrom` with `safeTransferFrom`

Factory.sol L115

```
 if (_baseToken == address(0)) {
            // transfer eth into the pool if base token is ETH
            address(privatePool).safeTransferETH(baseTokenAmount);
        } else {
            // deposit the base tokens from the caller into the pool
            ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);
        }
```

Replace `transferFrom` with `safeTransferFrom`, as some non-standard ERC20 tokens do not revert on failed transfers.