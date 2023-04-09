## Solmate’s SafeTransferLib doesn’t check whether the ERC20 contract exists
Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist.

Code snippet : https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L256
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L346
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L423
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L502

## Recommended Mitigation Steps
Add a contract exist control in functions;
```solidity
pragma solidity >=0.8.0;
function isContract(address _addr) private returns (bool isContract) {
    isContract = _addr.code.length > 0;
}
```