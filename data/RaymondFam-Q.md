## Missing setter for `changeFee`
`setChangeFee()` is missing in PrivatePool. In the event an adjustment is needed due to incorrect initialization, there is no option for that. Consider implementing a setter for `changeFee` just like it has been done so for its all other counterparts.

## Sanity checks at the constructor
Adequate zero address and zero value checks should be implemented in [`initialize()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157-L200) of PrivatePool to avoid accidental error(s) that could result in non-functional calls associated with it particularly when assigning presumably immutable variables, i.e. `baseToken`, `nft`, and `changeFee`.

## Inadequate NatSpec
Solidity contracts can use a special form of comments, i.e., the Ethereum Natural Language Specification Format (NatSpec) to provide rich documentation for functions, return variables and more. Please visit the following link for further details:

https://docs.soliditylang.org/en/v0.8.16/natspec-format.html

Consider fully equipping all contracts with complete set of NatSpec to better facilitate users/developers interacting with the protocol's smart contracts.

For instance, the function NatSpec of [`sell()`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L291-L306) in PrivatePool will be perfect with missing @return protocolFeeAmount added.  

## Unspecific compiler version pragma
For some source-units the compiler version pragma is very unspecific, i.e. ^0.8.19. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up actually checking a different EVM compilation that is ultimately deployed on the blockchain.

Avoid floating pragmas where possible. It is highly recommend pinning a concrete compiler version (latest without security issues) in at least the top-level “deployed” contracts to make it unambiguous which compiler version is being used. Rule of thumb: a flattened source-unit should have at least one non-floating concrete solidity compiler version pragma.

## Non-compliant contract layout with Solidity's Style Guide
According to Solidity's Style Guide below:

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

In order to help readers identify which functions they can call, and find the constructor and fallback definitions more easily, functions should be grouped according to their visibility and ordered in the following manner:

constructor, receive function (if exists), fallback function (if exists), external, public, internal, private

And, within a grouping, place the `view` and `pure` functions last.

Additionally, inside each contract, library or interface, use the following order:

type declarations, state variables, events, modifiers, functions

Consider adhering to the above guidelines for all contract instances entailed.

## Code repeatedly used should be grouped into a modifier
Grouping similar/identical code block into a modifier makes the code base more structured while reducing the contract size.

Here are the four instances entailed:

[File: EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

```solidity
101:        if (block.timestamp > deadline && deadline != 0) {
102:            revert DeadlinePassed();
103:        }

154:        if (block.timestamp > deadline && deadline != 0) {
155:            revert DeadlinePassed();
156:        }

228:        if (block.timestamp > deadline && deadline != 0) {
229:            revert DeadlinePassed();
230:        }

256:        if (block.timestamp > deadline && deadline != 0) {
257:            revert DeadlinePassed();
258:        }
```