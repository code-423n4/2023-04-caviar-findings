# Caviar QA report

# Low Issues

## L-01 Solmate’s SafeTransferLib doesn’t check whether the ERC20 contract exists

### Proof of Concept

[PrivatePool.sol L30](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L30)

[Factory.sol L27](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L27)
[EthRouter.sol L32](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L32)

Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).

This is stated in the Solmate library: https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

### Recommendation

Add a contract exist control in functions

## L-02 Low level calls don’t check for contract existence

### Proof of Concept

Low level calls return success if called on a destructed contract. See OpenZeppelin’s Address.so which checks address.code.length

There is 1 instance of this issue:

[PrivatePool.execute()](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L459-L476)

```
    function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
        // call the target with the value and data
        (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

        // if the call succeeded return the return data
        if (success) return returnData;

        // if we got an error bubble up the error message
        if (returnData.length > 0) {
            // solhint-disable-next-line no-inline-assembly
            assembly {
                let returnData_size := mload(returnData)
                revert(add(32, returnData), returnData_size)
            }
        } else {
            revert();
        }
    }

```

If the `target` address doesn't exist, the call will return true and the function won't revert.

### Recommendation

Check before any low-level call that the address actually exists, for example before the low level call in the callERC20 function you can check that the address is a contract by checking its code size.

## L-03 EthRouter.sol constructor lacking zero address check

### Proof of Concept

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L90-L92

### Recommendation

The `royalRegisrty` address is an immutable one once set, advisably minimal checks like the zero address check should be implemented to ensure that the address does not mistakenly get set to a wrong one.

## L-04 Use `safeTransferOwnership` instead of `transferOwnership` function

### Proof of Concept

[Factory.sol L26](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L26)
[Factory.sol L37](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L37)
transferOwnership function is used to change Ownership from Owned.sol.

Use a two structure transferOwnership which is safer.
safeTransferOwnership,is known to be more secure due to its two-stage ownership transfer.

### Recommendation

Use [Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)

## L-05 `EthRouter.buy/sell()` possible OOG error

### Proof of Concept

[EthRouter.buy()](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L99-L144)
[EthRouter.sell()](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L152-L209)

### Recommendation

Both functions takes an array and makes a lot of computation on each iteration, a length limit should be provided to the passed in array length

## L-06 `Factory.sol` missing Zero address checks

### Proof of Concept

Multiple instances within contracts in scope:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L132

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135-L137

### Recommendation

Lack of zero address checks procedure for critical operations leaves them error-prone. Consider adding zero address checks on the critical functions.

## L-07 Using array memory parameter without checking its length

### Proof of Concept

Multiple instances in code:

[PrivatePool.change()](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L385-L452)

[PrivatePool.sumWeightsAndValidateProof()](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L661-L687)

These array memory parameter can be problematic if not used properly , if the array is very large it may overlap over other part of memory.

### Recommendation

check array length before using it

## L-08 Incomplete input validation at `PrivatePool.setVirtualReserves()`

### Proof of Concept

[`PrivatePool.setVirtualReserves()`](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L545)

### Recommendation

Minimal checks should be added to check that the virtual reserves shouldn't be set to 0

## L-09 Incomplete input validation at `PrivatePool.initialize()` and `constructor`

### Proof of Concept

[constructor]
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L157-L200)

[PrivatePool.initialize()](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L157-L200)

### Recommendation

Some variables are immutable after being set in the constructor/initializer, so additional checks like zero address, defaulting to 0, would produce a leap in protective layers.
