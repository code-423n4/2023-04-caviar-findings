Title: Consider delete empty block or emit something

impact:
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.

Proof of Concept:
[Factory.sol#L55](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L55)
[EthRouter.sol#L88](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88)
________________________________________________________________________

Title: Move `SafeTransferLib` and `LibClone` functions into the `PrivatePool` contract to avoid unnecessary function calls and gas costs.

Proof of Concept:
Instead of using SafeTransferLib and LibClone in the Factory contract, we can move these functions into the PrivatePool contract and call them directly from there. This can help to reduce the number of function calls and gas costs.
[Factory.sol#L38-L39](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L38-L39)
________________________________________________________________________


Title: Use `uint256` instead of `uint128` to avoid unnecessary conversions and gas costs.

Proof of Concept:
[Factory.sol#L74-L75](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L74-L75)
________________________________________________________________________


Title: function `sumWeightsAndValidateProof()` gas improvement on returning value

Proof of Concept:
[PrivatePool.sol#L671](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L671)

Recommended Mitigation Steps:
by set `sum` in returns L#665 and delete L#671 can save gas

```
function sumWeightsAndValidateProof(
        uint256[] memory tokenIds,
        uint256[] memory tokenWeights,
        MerkleMultiProof memory proof
    ) public view returns (uint256 sum) { //@audit-info: set here
```
________________________________________________________________________

Title: Use `require` instead of `revert`

Proof of Concept:
Use require instead of revert in the create function. require has a lower gas cost and will automatically refund any gas used up to the point where the error occurred.
[Factory.sol#L87-L89](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87-L89)

Recommended Mitigation Steps:
```
require(
            (_baseToken == address(0) && msg.value != baseTokenAmount) ||
            (_baseToken != address(0) && msg.value > 0),
            "Invalid ETH amount"
        );
```
________________________________________________________________________