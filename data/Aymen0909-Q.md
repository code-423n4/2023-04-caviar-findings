# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1 | Possible loss of `msg.value` when calling the `flashloan` function | Low | 1 |
| 2 | slippage protection should be included in `sell` function | Low | 1 |
| 3 | No zero address check on `royaltyRecipient` in EthRouter | Low | 2 |
| 4 | `protocolFeeRate` should have a maximum upper bound | Low | 1 |
| 5 | Immutable state variables lack zero address checks | Low | 4 |
| 6 | Event should be emitted in setters | NC | 3 |
| 7 | Avoid floating pragma where possible | NC |  |
| 8 | `public` functions not called by the contract should be declared `external` instead | NC | 20 |

## Findings

### 1- Possible loss of `msg.value` when calling the `flashloan` function :

#### Risk : Low

When the `flashloan` function is called by a user it allows both the ETH and ERC20 tokens payments, so when the `baseToken` is an ERC20 token the user will pay with ERC20 tokens but if he also sends ETH in the form of `msg.value` in the same transaction (by accident), this ETH fund will be lost as the `flashloan` function doesn't verify that `msg.value == 0` when baseToken is an ERC20 tokens.

The impact of this is that those ETH funds sent by accident will be locked in the `PrivatePool` contract and the user won't be able to get them back.

The issue occurs because the `flashloan` function just checks that `msg.value >= fee` in case  `baseToken == address(0)` but does not that `msg.value == 0` when `baseToken != address(0)`, as it can be seen in the code below :

File: PrivatePool.sol [Line 635](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635)
```
if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
```

#### Mitigation
To avoid this issue the check in the `flashloan` function should verify that `msg.value == 0` when `baseToken != address(0)`, thus the check [Line 635](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635) should be modified as follows :

```
if ((baseToken == address(0) && msg.value < fee) || (baseToken != address(0) && msg.value > 0)) revert InvalidEthAmount();
```

### 2- Slippage protection should be included in `sell` function :

#### Risk : Low

According to the comments of the `sell` function in the `PrivatePool` contract the slippage protection is not guaranted and must be done by the users using a wrapper contract, this very unconvenient for the users as they are forced to create a new contract and implement a selling logic with slippage protection instead for selling directly with a simple call.

This issue can be avoided by including the slippage protection directly into the `sell` function by adding a minAmountOut argument to the function and checking the actual amountOut against it. Note that this solution can be implemented without massive changes to the `sell` function as it is illustrated in the code below :

```
function sell(
    uint256[] calldata tokenIds,
    uint256[] calldata tokenWeights,
    uint256 minAmountOut,
    MerkleMultiProof calldata proof,
    IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
    // ~~~ Checks ~~~ //

    // calculate the sum of weights of the NFTs to sell
    uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

    // calculate the net output amount and fee amount
    (netOutputAmount, feeAmount, protocolFeeAmount) = sellQuote(weightSum);

    // @audit slippage protection
    if (netOutputAmount < minAmountOut) revert TooHighSlippage();

    ...
}
```

#### Mitigation
You should considere including the slippage protection directly into the `sell` function.

### 3- No zero address check on `royaltyRecipient` in EthRouter :

#### Risk : Low

In the `buy` and `sell` functions inside the `EthRouter` contract there is no non-zero address check for the `royaltyRecipient` address when trying to pay royalties, so ETH funds can be burned (sent to address(0)) when trying to trade.

Instances of this issue :

File: EthRouter.sol [Line 118-121](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L118-L121)

File: EthRouter.sol [Line 185-188](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L185-L188)

#### Mitigation
Add a non zero address check for the `royaltyRecipient` address in both `buy` and `sell` functions.


### 4- `protocolFeeRate` should have a maximum upper bound  :

#### Risk : Low

The value of `protocolFeeRate` is in basis point of 10000 (meaning that 10000≈100%) but in the fee setter function `setProtocolFeeRate` there is no check to ensure that the fee is not above 10000, which means that the owner can set it to a large value (even greater than 10000).The consequence of this can be : 

* If `protocolFeeRate > 10000` Some functions like `buyQuote` or `sellQuote` will be blocked due to an undeflow error which will result in the DOS of the buy & sell functions.

* If `protocolFeeRate` is large but less than 10000 users will pay a very large fee when trading on the private pools.

#### Mitigation
You should considere adding an upper bound value when setting `protocolFeeRate`.


### 5- Immutable state variables lack zero address checks :

Constructors should check the values written in an immutable state variables(address) is not the address(0).

#### Risk : Low

#### Proof of Concept
Instances include:

File: PrivatePool.sol [Line 144-146](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L144-L146)
```
factory = payable(_factory);
royaltyRegistry = _royaltyRegistry;
stolenNftOracle = _stolenNftOracle;
```

File: EthRouter.sol [Line 91](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L91)
```
royaltyRegistry = _royaltyRegistry;
```

#### Mitigation
Add non-zero address checks in the constructors for the instances aforementioned.

### 6- Event should be emitted in setters :

The setter functions which are responsible of critical changes to the protocol working should emit an event so that users and others parties are acknowledged of this changes and thus can adapte to them. 

#### Risk : Non critical

#### Proof of Concept

Instances include:

File: Factory.sol

[function setPrivatePoolMetadata(address _privatePoolMetadata)](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129)

[function setPrivatePoolImplementation(address _privatePoolImplementation)](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135)

[function setProtocolFeeRate(uint16 _protocolFeeRate)](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141)

#### Mitigation
Emit an event in all the aforementioned setters.

### 7- Avoid floating pragma where possible :

#### Risk : Non critical

All the contracts of the protocol use a floating solidity version `pragma solidity ^0.8.19` which should be fixed to agiven version (preferably to a recent one) to avoid any issues in the future, as locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package.

### 8- `public` functions not called by the contract should be declared `external` instead :

#### Risk : Non critical

The `public` functions that are not called inside the contract should be declared `external` instead.

There are 20 instances of this issue:

```
File: PrivatePool.sol

211     function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof) public
301     function sell(
            uint256[] calldata tokenIds,
            uint256[] calldata tokenWeights,
            MerkleMultiProof calldata proof,
            IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
        ) public
385     function change(
            uint256[] memory inputTokenIds,
            uint256[] memory inputTokenWeights,
            MerkleMultiProof memory inputProof,
            IStolenNftOracle.Message[] memory stolenNftProofs,
            uint256[] memory outputTokenIds,
            uint256[] memory outputTokenWeights,
            MerkleMultiProof memory outputProof
        ) public
459     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory)
484     function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable
514     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public
602     function setAllParameters(
            uint128 newVirtualBaseTokenReserves,
            uint128 newVirtualNftReserves,
            bytes32 newMerkleRoot,
            uint16 newFeeRate,
            bool newUseStolenNftOracle,
            bool newPayRoyalties
        ) public
742     function price() public view returns (uint256)
755     function flashFeeToken() public view returns (address)

File: PrivatePoolMetadata.sol

17  	function tokenURI(uint256 tokenId) public view returns (string memory)

File: EthRouter.sol

99      function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public
152     function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public
219     function deposit(
            address payable privatePool,
            address nft,
            uint256[] calldata tokenIds,
            uint256 minPrice,
            uint256 maxPrice,
            uint256 deadline
        ) public
254     function change(Change[] calldata changes, uint256 deadline) public

File: Factory.sol

129     function setPrivatePoolMetadata(address _privatePoolMetadata) public
135     function setPrivatePoolImplementation(address _privatePoolImplementation) public 
141     function setProtocolFeeRate(uint16 _protocolFeeRate) public
148     function withdraw(address token, uint256 amount) public 
161     function tokenURI(uint256 id) public view override returns (string memory) 
168     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress)
```