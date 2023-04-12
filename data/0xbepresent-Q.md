1 - There is a discrepancy between the documentation and the code in the ```PrivatePool.change()``` function
==

The documentaion about the [change()](https://docs.caviar.sh/technical-reference/custom-pools/smart-contract-api/privatepool#change) function says: *The sum of the caller's NFT weights must be less than or equal to the sum of the output pool NFTs weights.* Additionally in the [next code comment](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L376) it says: *The sum of the caller's NFT weights must be less than or equal to the sum of the output pool NFTs weights.*

The problem is that the code is totally the opposite, the code will revert [if the sum of the caller's NFT weights is less.](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L413)

```solidity
if (inputWeightSum < outputWeightSum) revert InsufficientInputWeight();
```

Fix the discrepancy between the code and the documentation.

2 - The flashLoan fee token should use the ```flashFeeToken()``` to get the fee token
==

The flash loan fee token used in the [next line](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L651):

```solidity
if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);
```

It uses the ```baseToken``` for the fee token but it should use the [flashFeeToken()](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L755) function instead to get the flash fee token.

3 - Add a function that helps to update the ```changeFee```
==

The [changeFee](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L179) is initialized in the beginning and it can not be updated in the future by the private pool owner.

Add a ```updateChangeFee()``` function that helps to update the ```changeFee``` if it is necessary.

4 - In the ```Factory.create()``` function the ```PrivatePool.deposit()``` can be used instead, for the base token and NFTs deposit.
==

The ```Factory.create()``` function [uses the next code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L110-L121) that helps to deposit the ```baseToken``` and the NFTs. It can use the ```privatePool.deposit()``` instead and the code will be cleaner.