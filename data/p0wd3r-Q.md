# Check if tokenIds is empty at the beginning of the buy function.

PrivatePool.sol L211

```
function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof) {
...
emit Buy(tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);
}
```

The `buy` function does not check if `tokenIds` is empty. If it is empty, the function will continue to execute and eventually emit an event that has no effect and should not be emitted.

# The parameters of flashFee were not used.

PrivatePool.sol L750

```
function flashFee(address, uint256) public view returns (uint256) {
    return changeFee;
}
```

# import IERC3156FlashBorrower.sol indead of IERC3156FlashLender.sol
PrivatePool.sol L34

```
import {IERC3156FlashBorrower} from "openzeppelin/interfaces/IERC3156FlashLender.sol";
```

It only use IERC3156FlashBorrower, other functions in IERC3156FlashLender.sol are not used.