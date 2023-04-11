## possible address conflict

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L92
```
        privatePool = PrivatePool(payable(privatePoolImplementation.cloneDeterministic(_salt)));
```


This contract uses the cloneDeterministic() function from the LibClone library to create a cloned copy of the minimal proxy contract, which computes a deterministic address. However, there may be potential risks and issues if the computed address is already occupied by someone else.
## Code readability
This smart contract uses multiple nested conditional statements and loops, making the code complex and hard to understand. This could lead to errors by developers while writing or modifying the code, thus increasing the risk of vulnerabilities.
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## Token recovery not implemented
The contract does not have a token recovery mechanism in place. If users accidentally send tokens to the contract address, these tokens will be permanently lost and cannot be retrieved from the contract.
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## Unconsidered reentrancy attack
If a reentrant attack occurs before the receiver.onFlashLoan() function is executed in the flashLoan function, it may result in multiple calls to the onFlashLoan() function and potential fund loss. It is recommended to use OpenZeppelin's ReentrancyGuard or similar protection measures to prevent such attacks.
 In the _getRoyalty function, if the lookupAddress returned by the royaltyRegistry smart contract does not implement the ERC-2981 interface, it will result in uninitialized recipient and royaltyFee variables being returned by the function. This may allow malicious attackers to call this function through reentrancy attacks or other means and potentially lead to fund loss or other vulnerabilities. It is recommended to add an extra check to ensure that the lookupAddress implements the correct interface.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L623
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L778









