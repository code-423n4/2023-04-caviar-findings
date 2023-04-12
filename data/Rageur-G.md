## GAS-1: <X> += <Y> costs more gas than <X> = <X> + <Y> for state variables

### Description

Using the addition operator instead of plus-equals saves gas.

### Affected file

* PrivatePool.sol (Line: 230, 231, 323, 324)

## GAS-2: Functions guaranteed to revert when called by normal users can be marked payable

### Description

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

### Affected file

* Factory.sol (Line: 129, 135, 141, 148)
* PrivatePool.sol (Line: 514, 538, 550, 562, 576, 587)

## GAS-3: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* EthRouter.sol (Line: 99, 219, 254)
* Factory.sol (Line: 71, 129, 135, 141, 148, 168)
* PrivatePool.sol (Line: 157, 211, 301, 385, 459, 484, 514, 602, 742, 755)
* PrivatePoolMetadata.sol (Line: 17)

## GAS-4: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* PrivatePool.sol (Line: 127)

## GAS-5: Usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

### Description

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.

### Affected file

* Factory.sol (Line: 71, 71, 71, 141)
* PrivatePool.sol (Line: 157, 230, 231, 323, 324, 538, 562, 602)

## GAS-6: Use selfbalance()

### Description

Use selfbalance() instead of address(this).balance when getting your contract’s balance of ETH to save gas.

### Affected file

* EthRouter.sol (Line: 141, 142, 203, 208, 290, 291)

## GAS-7: Using immutable on variables that are only set in the constructor and never after (2.1k gas per var)

### Description

Use immutable if you want to assign a permanent value at construction. Use constants if you already know the permanent value. Both get directly embedded in bytecode, saving SLOAD.
Variables only set in the constructor and never edited afterwards should be marked as immutable, as it would avoid the expensive storage-writing operation in the constructor (around 20 000 gas per variable) and replace the expensive storage-reading operations (around 2100 gas per reading) to a less expensive value reading (3 gas).

### Affected file

* EthRouter.sol (Line: 86)
* PrivatePool.sol (Line: 119, 122, 125)