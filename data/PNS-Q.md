# Low Risk

## Missing checks for `address(0x0)` when assigning values to address state variables

```solidity
File: src/EthRouter.sol
90:     constructor(address _royaltyRegistry) {
91:         royaltyRegistry = _royaltyRegistry;
92:     }
```
```solidity
File: src/PrivatePool.sol
143:     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
144:         factory = payable(_factory);
145:         royaltyRegistry = _royaltyRegistry;
146:         stolenNftOracle = _stolenNftOracle;
147:     }
```
```solidity
File: src/PrivatePool.sol
175:         baseToken = _baseToken;
176:         nft = _nft;
```

## Consider `isContract` checks in constructors
Several addresses are assigned in the contract constructors and assigned to immutable variables. A successful deployment is sensitive to these addresses being assigned correctly for the current network, and that addresses were specified in the correct order. Consider adding checks, as aggressively as possible for the use case, to help ensure the deployment configuration is correct.
`.isContract()` is referring to the [OZ Address library](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/Address.sol#L40) or similar implementation.

```solidity
File: src/EthRouter.sol
90:     constructor(address _royaltyRegistry) {
91:         royaltyRegistry = _royaltyRegistry;
92:     }
```
```solidity
File: src/PrivatePool.sol
143:     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
144:         factory = payable(_factory);
145:         royaltyRegistry = _royaltyRegistry;
146:         stolenNftOracle = _stolenNftOracle;
147:     }
```

# Non-Critical

## `public` functions not called by the contract should be declared `external` instead
```solidity
File: src/EthRouter.sol
99:     function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
...
152:     function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public {
...
219:     function deposit(
...
254:     function change(Change[] calldata changes, uint256 deadline) public payable {
```

## Common code should be refactored

The code snippet below appears in four functions `buy`, `sell`, `deposit`, `change` in the contract `EthRouter.sol` and can be replaced by a modifier.

```solidity
File: src/EthRouter.sol
100:         // check that the deadline has not passed (if any)
101:         if (block.timestamp > deadline && deadline != 0) {
102:             revert DeadlinePassed();
103:         }
```

## Function ordering does not follow the Solidity style guide
According to the [Solidity style guide](https://docs.soliditylang.org/en/v0.8.13/style-guide.html#order-of-functions), functions should be laid out in the following order:
- constructor
- receive function (if exists)
- fallback function (if exists)
- external
- public
- internal
- private

`EthRouter.sol`, `PrivatePool.sol` do not meet the guidelines.

## Spellcheck
Consider using the [VSCode plugin](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker) or similar to help catch spelling errors during development.

Example:
`exceute` => `execute`
`aount` => `amount`
```solidity
File: src/EthRouter.sol
169:                 // exceute the sell against a public pool

File: src/PrivatePool.sol
251:         // add the royalty fee amount to the net input aount
```
