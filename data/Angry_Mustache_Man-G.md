## [G-1]-Structs can be packed into fewer storage slots

In 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L48-L56
and In 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L58-L67

'bool isPublicPool' can be moved upwards to 'address nft' , which could have saved one storage slot and thereby saving gas.
 
## [G-2] Duplicated require()/if() checks should be refactored to a modifier or function
(1)
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L254

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L421

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L500

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651

(2)
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L242

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L271

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L333

(3) 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256

## [G-3] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contracts gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html

Use a larger size then downcast where needed.

## [G-4] Use nested if and, avoid multiple check combinations

Using nested is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

### Occurence

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256

## [G-5] Functions guaranteed to revert_ when called by normal users can be marked payable

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Occurences
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129

## [G-6] Setting the constructor to payable
You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L53

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143

## [G-7] Use assembly to write address storage values

Use assembly language while writing to storage- which usually happen in contructors, so as to save some gas.

## [G-8] x += y (x -= y) costs more gas than x = x + y (x = x - y) for state variables.

You can replace all -= and += occurrences to save gas

## [G-9] Upgrade Solidity’s optimizer
Make sure Solidity’s optimizer is enabled. It reduces gas costs. If you want to gas optimize for contract deployment (costs less to deploy a contract) then set the Solidity optimizer at a low number. If you want to optimize for run-time gas costs (when functions are called on a contract) then set the optimizer to a high number.


Set the optimization value higher than 800 in your foundry.toml file.

## [G-10] Amounts should be checked for 0 before calling a transfer

Checking non-zero transfer values can avoid an expensive external call and save gas.

