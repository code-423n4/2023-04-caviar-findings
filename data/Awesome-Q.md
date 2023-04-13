# 1. Unspecific Compiler Version Pragma

It is generally not recommended to use floating pragmas (i.e. pragmas that do not specify a specific compiler version) in contracts that are not intended to be used as libraries.

This is because using floating pragmas in application contracts can pose a security risk.

For example, a known vulnerable compiler version may be selected by mistake, or security tools might revert to an older compiler version that produces a different EVM compilation than the one intended to be deployed on the blockchain.

To avoid these potential issues, consider specifying a specific compiler version in your pragmas.

So instead of using a floating pragma like `pragma solidity ^0.8.0;`, it is better to use a concrete compiler version like `pragma solidity 0.8.17;`.

More information can be found in the following links:

- [Consensys Audit of 1inch](https://consensys.net/diligence/audits/2020/12/1inch-liquidity-protocol/#unspecific-compiler-version-pragma)
- [Solidity docs](https://docs.soliditylang.org/en/latest/layout-of-source-files.html#version-pragma)
- [Solidity Specific](https://consensys.github.io/smart-contract-best-practices/development-recommendations/solidity-specific/locking-pragmas/)

Affected lines of code:

- [Factory.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L2)
- [PrivatePoolMetadata.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L2)
- [EthRouter.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L2)
- [PrivatePool.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L2)

# 2. Follow the function order of the solidity style guide

The [Solidity style guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) recommends the following function order:

- constructor

- receive function (if exists)

- fallback function (if exists)

- external

- public

- internal

- private

Within a grouping, place the `view` and `pure` functions last.

This is because "Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier." -solidity style guide

Affected file:
[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

# 3. Use Modifier For Validations Used Repeatedly

To improve the maintainability and reusability of code, it's best to group repeated validations into a modifier. By using modifiers, you can make your code more organized and easier to read, while also reducing the overall code size.

Affected lines of code:

- [Lines 101-103](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L101-L103)
- [Lines 154-156](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L154-L156)
- [Lines 228-230](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L228-L230)
- [Lines 256-258](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L256-L258)

# 4. Fix typos to improve readability

```solidity
File: src/EthRouter.sol
line 169:    // exceute the sell against a public pool
```

Consider making the following change to [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol):

- "exceute" To "execute" ([Line 169](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L169))

```
File: src/PrivatePool.sol
line 251: // add the royalty fee amount to the net input aount
```

Consider making the following changes to [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol):

- "aount" To "amount" ([Line 251](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L251))

# 5. Add a Setter for `changeFee` in PrivatePool Contract

The PrivatePool contract is missing a `setChangeFeeAmount()` function, which can be problematic if an adjustment is needed due to incorrect initialization. Therefore, it's best to consider implementing a setter function for `changeFee`.

Overall adding a setter function for `changeFee`, you can provide a workaround in case the initial fee is set incorrectly or needs to be adjusted later on. This can improve the flexibility and robustness of the contract.

Here's an example of how to implement a `setChangeFeeAmount(`)` function in the PrivatePool contract:

```solidity
function setChangeFeeAmount(uint256 newFee) external onlyOwner {
    changeFee = newFee;
}
```
