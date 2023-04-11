# 1: FLOATING PRAGMA

Vulnerability details

## Context:

All the contracts in scope are floating the pragma version.


## Proof of Concept

All contracts in scope

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case of contracts in a library or a package.


# 2: PRAGMA VERSION ^0.8.19 VERSION TOO RECENT TO BE TRUSTED.

Vulnerability details

## Context:

Unexpected bugs can be reported in recent versions;
Risks related to recent releases
Risks of complex code generation changes
Risks of new language features
Risks of known bugs

0.8.19 (2023-02-22)
0.8.18 (2023-02-01)
0.8.17 (2022-09-08)
0.8.16 (2022-08-08)

For reference, see https://github.com/ethereum/solidity/blob/develop/Changelog.md  


## Proof of Concept

 > ***File: Factory.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L2


 > ***File: PrivatePoolMetadata.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L2


 > ***File: EthRouter.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L2


 > ***File: PrivatePool.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L2


> ***File: IStolenNftOracle.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/interfaces/IStolenNftOracle.sol#L2

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Use a non-legacy and more battle-tested version


# 3: ADD A LIMIT FOR THE MAXIMUM NUMBER OF CHARACTERS PER LINE

Vulnerability details

## Context:

The solidity documentation recommends a maximum of 120 characters.

For reference, see https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length

## Proof of Concept

> ***2 Files, 6 Instances*** 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L47

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L103

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L58-L60

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L63

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider adding a limit of 120 characters or less to prevent large lines.


# 4: GENERATE PERFECT CODE HEADERS EVERY TIME					

Vulnerability details

## Context:

We recommend using a header for Solidity code layout and readability

For reference, see https://github.com/transmissions11/headers

## Proof of Concept

All Contracts

 /*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/

### Tools Used

Manual Analysis


# 5: IMPORTS CAN BE GROUPED TOGETHER

Vulnerability details

## Context:

Imports can be grouped together.

## Proof of Concept

> ***File: EthRouter.sol*** 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L31-L38


> ***File: PrivatePool.sol***

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L27-L37

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Consider importing OZ first, then all interfaces, then all utils.


# 6: ADD TO BLACKLIST FUNCTION

Vulnerability details

## Context:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, We recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported for theft, and NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code
	
## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Add to Blacklist function and modifier.

> ***Example***

    modifier nonBlacklistRequired(address extension) {
        require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
        _;
    }


# 7: WORD & TYPING TYPOS

Vulnerability details

## Context:

Word & typing typos

## Proof of Concept

> ***File: PrivatePool.sol*** 

aount can be changed to amount in the following comment.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L251


nft can be changed to NFT in the following comments.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L666

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L106

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L315

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L330

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L406

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L409

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L440

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L445

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L495

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L517

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L539

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L666


### Tools Used

Manual Analysis


# 8: EVENT IS MISSING INDEXED FIELDS

Vulnerability details

## Context:

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields and gas usage is not, particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

## Proof of Concept

> ***Instances: (9)***

> ***File: PrivatePool.sol***

59:    event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
60:    event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
61:    event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);

63:    event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
64:    event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);
65:    event SetMerkleRoot(bytes32 merkleRoot);
66:    event SetFeeRate(uint16 feeRate);
67:    event SetUseStolenNftOracle(bool useStolenNftOracle);
68:    event SetPayRoyalties(bool payRoyalties);

[Link to code](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

## Tools Used

Manual Analysis

### Recommended Mitigation Steps

Events should be indexed where necessary 


# 9: FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE

Vulnerability details

## Context:

Order of Functions; ordering helps readers identify which functions they can call and find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

For reference, see https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## Proof of Concept

> ***Functions should be grouped according to their visibility and ordered:***

-constructor
-receive function (if exists)
-fallback function (if exists)
-external
-public
-internal
-private
-within a grouping, place the view and pure functions last

### Tools Used

Manual Analysis


X