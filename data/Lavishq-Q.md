## 1. The versions of solidity is the latest with floating pragma
version ^0.8.19 is used in the project and it is the most recent solidity version. Apart from that the files are have floating pragma and that could lead to deployment with v0.8.20 when it is released and there are always a slight possibility of bugs and errors in the latest versions. So, it is better to use more tested versions like 0.8.17.

## 2. Single step ownership transfer

Transferring Ownership in a single transfer transaction in worst cases could hand over the ownership to an arbitrary address by a simple mistake and is not reversible. So it is recommended to use 2 step ownership transfer from openzeppelin library

[Ownable2Step](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)


## 3. The nested for loop on array could lead to failure of tx
[buy()](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L99) and [sell()](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L152) functions loops through array of buy orders and sell orders. In case, the array is size is large, the function could consume a significant amount of gas and fail to execute within the gas limit, potentially resulting in a loss of gas.

## 4. Address 0 check is not performed on nft address
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L71

The create function creates a pool Pair without even checking the address 0 on the nft

## 5. The contract should use timelock for changing fee
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141
It should emit an event and it definitely needs to have timelock for changing fee or setting fee for the protocol. The reason is as a user no one would like if the admin has the power to set/change fee at his whims. So, it is best to use [timelock](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/governance/TimelockController.sol). 

## 6. check the array length private pool
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L661

It is necesary to check array length in cases where there is use of for loop on arrays so that it doesn't result in unwanted consequence or error (array out of bounds)

## 7. functions can be external instead of public
All the functions in the contract are public and most of them can be changed to external or sometimes internal/private. So, it is recommended and it will save gas during deployment and for user during transaction.