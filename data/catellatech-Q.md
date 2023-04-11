`Dear Caviar Team, as we have gone through each contract within the scope, we have noticed good practices that have been implemented. However, we have identified some inconsistencies that we recommend addressing. We understand that every team has a different level of good practices, but we believe that at least 90% of the recommendations in the following report should be applied for better gas efficiency, readability, and most importantly, safety.`

`Note: We have provided a description of the situation and recommendations to follow, including articles and resources we have created to help identify the problem and address it quickly, and to implement them in future projects.`

### Issues
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |

| Total Found Issues | 20 |
|:--:|:--:|

### Low Risk
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | ADD A TIMELOCK TO CRITICAL FUNCTIONS | 1 |
| [L-02] | IN THE EVENTS, INCLUDE THE OLD AND NEW VALUES OF THE UPDATED PARAMETERS TO TRACK THE CHANGES MADE  | 3 |
| [L-03] | MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS | 3 |
| [L-04] | USE THE OPENZEPPELIN SAFECAST LIBRARY FOR CRITICAL FUNCTIONS | 2 |

| Total Low Issues | 4 | Total Instances | 9 |
|:--:|:--:|:--:|--:|

### Non-Critical
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | USE OF FLOATING PRAGMA | all contracts |
| [N-02] | MANDATORY CHECKS FOR EXTRA SAFETY IN THE SETTERS  | 2 |
| [N-03] | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE | 1 |
| [N-04] | MISSING EMERGENCY STOP (CIRCUIT BREAKER) PATTERN | 4 |
| [N-05] | TAKE ADVANTAGE OF CUSTOM ERROR'S RETURN VALUE PROPERTY | all the contracts |
| [N-06] | USE THE LATEST UPDATED VERSION OF OPENZEPPELIN DEPENDENCIES | |
| [N-07] | NEED FUZZING TEST | all contracts |
| [N-08] | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED() | 7 |
| [N-09] | LINES ARE TOO LONG | 4 |
| [N-10] | SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE | 12 |
| [N-11] | CONTRACT DOES NOT FOLLOW THE SOLIDITY STYLE GUIDE'S SUGGESTED LAYOUT ORDERING | 2 |
| [N-12] | ASSEMBLY CODES SPECIFIC - SHOULD HAVE COMMENTS | 1 |

| Total Non-Critical Issues | 12 | Total Instances | 33 |
|:--:|:--:|:--:|--:|

### Refactor Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | FUNCTION NAMING SUGGESTIONS | 1 |
| [R-02] | SHORTHAND WAY TO WRITE IF/ELSE STATEMENT | 6 |
| [R-03] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO) | 9 |

| Total Refactor Issues | 3 | Total Instances | 16 |
|:--:|:--:|:--:|--:|


# Detailed Findings

## [L-01] ADD A TIMELOCK TO CRITICAL FUNCTIONS
It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to the following functions:

### PROOF OF CONCEPT
```solidity
main/src/Factory.sol

141:  function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner
```

## [L-02] IN THE EVENTS, INCLUDE THE OLD AND NEW VALUES OF THE UPDATED PARAMETERS TO TRACK THE CHANGES MADE
The following functions Factory.setProtocolFeeRate make critical changes and should include the `old and new values` of the updated parameters so that users can be aware of the changes made.

### MITIGATION
The example of how to mitigate the problem we raised:
```diff
- 225:  emit ChangeMaxAmount(maxAmount);
+ 225:  emit ChangeMaxAmount(oldMaxAmount, newMaxAmount);
```

## [L-03] MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS
The afunctions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

missing events and timelocks do not promote transparency and if such changes immediately affect users' perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.


### PROOF OF CONCEPT
```solidity
main/src/Factory.sol

// @audit these functions should have events
    129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner
    135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner 
    141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner
```

## [L-04] USE THE OPENZEPPELIN SAFECAST LIBRARY FOR CRITICAL FUNCTIONS
We have noticed that the contract `PrivatePool.sol.` implements many type conversions from uint256 to uint128. We recommend using the OpenZeppelin SafeCast library to make the project more robust and take advantage of the gas optimizations and best practices provided by OpenZeppelin.

### PROOF OF CONCEPT
```solidity
main/src/PrivatePool.sol

211: function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
    {
        // ~~~ Checks ~~~ //

        // calculate the sum of weights of the NFTs to buy
        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

        // calculate the required net input amount and fee amount
        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

        // check that the caller sent 0 ETH if the base token is not ETH
        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

        // ~~~ Effects ~~~ //

        // update the virtual reserves
        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount); // @audit Implement safe cast 
        virtualNftReserves -= uint128(weightSum); // @audit Implement safe cast 
    }

301: function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
        // ~~~ Checks ~~~ //

        // calculate the sum of weights of the NFTs to sell
        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

        // calculate the net output amount and fee amount
        (netOutputAmount, feeAmount, protocolFeeAmount) = sellQuote(weightSum);

        //  check the nfts are not stolen
        if (useStolenNftOracle) {
            IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
        }

        // ~~~ Effects ~~~ //

        // update the virtual reserves
        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount); // @audit Implement safe cast 
        virtualNftReserves += uint128(weightSum); // @audit Implement safe cast 
```
### RECOMMENDATION
```solidity
openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol#L290
    /**
     * @dev Returns the downcasted uint128 from uint256, reverting on
     * overflow (when the input is greater than largest uint128).
     *
     * Counterpart to Solidity's `uint128` operator.
     *
     * Requirements:
     *
     * - input must fit into 128 bits
     *
     * _Available since v2.5._
     */
    function toUint128(uint256 value) internal pure returns (uint128) {
        require(value <= type(uint128).max, "SafeCast: value doesn't fit in 128 bits");
        return uint128(value);
    }
```

## [N-01] USE OF FLOATING PRAGMA
 It is advisable to avoid using floating pragmas in contracts that are not libraries.

While floating pragmas make sense in the context of libraries, allowing them to be included in multiple different versions of applications, they can pose a security risk in application implementations.

There is a possibility that a vulnerable compiler version may be accidentally selected or security tools may fallback to an older compiler version, resulting in verification of a different EVM compilation that is ultimately deployed on the blockchain.

Therefore, it is recommended to pin to a specific compiler version to ensure the security of the contract.

### PROOF OF CONCEPT
```solidity
// @audit all the contracts have this issue
```
### MITIGATION
```diff
- 2: pragma solidity ^0.8.19;
+ 2: pragma solidity 0.8.19;
```

## [N-02] MANDATORY CHECKS FOR EXTRA SAFETY IN THE SETTERS 
In the folowing functions below, there are some checks that can be made in order to achieve more safe and efficient code.

Address zero check can be added in the function offer, RubiconMarket contract to ensure the address owner and recipient aren't address(0).

### PROOF OF CONCEPT
```solidity
main/src/Factory.sol

129: function setPrivatePoolMetadata(address _privatePoolMetadata) // @audit Check _privatePoolMetadata it's not address 0
135: function setPrivatePoolImplementation(address _privatePoolImplementation) // @audit Check _privatePoolImplementation it's not address 0                        
```

## [N-03] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE
According to the Solidity style guide, functions should be laid out in the following order: 

- constructor() 
- receive() 
- fallback() 
- external
- public 
- internal 
- private

[Solidity Order Functions](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions)

Functions should be grouped according to their visibility and ordered: `within a grouping, place the view and pure functions last`

Apply this recommendation in the [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88) contract.

## [N-04] MISSING EMERGENCY STOP (CIRCUIT BREAKER) PATTERN
At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an `EMERGENCY STOP (CIRCUIT BREAKER) PATTERN`. Use the follow example to implement it into in the `EthRouter.sol`, in the following functions `buy`, `sell`,`change` and `deposit`.

[Emergency Stop Pattern Example](https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol)


## [N-05] TAKE ADVANTAGE OF CUSTOM ERROR'S RETURN VALUE PROPERTY
An important feature of [Custom Errors](https://docs.soliditylang.org/en/v0.8.19/abi-spec.html#errors) is that values such as `address`, `tokenID`, `msg.value` can be written inside the `()` sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

### PROOF OF CONCEPT
```solidity
main/src/PrivatePool.sol

// @audit so many instances with this observation here an example  
    225: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount()
    262: if (msg.value < netInputAmount) revert InvalidEthAmount();
    397: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
    429: if (msg.value < feeAmount + protocolFeeAmount) revert InvalidEthAmount();
```

## [N-06] USE THE LATEST UPDATED VERSION OF OPENZEPPELIN DEPENDENCIES
### PROOF OF CONCEPT
```json
package.json
"dependencies": {
    "@openzeppelin/merkle-tree": "^1.0.2",
}

```
### MITIGATION
Upgrade OpenZeppelin to version [1.0.4](https://github.com/OpenZeppelin/merkle-tree/compare/v1.0.2...v1.0.4?short_path=06572a9#diff-06572a96a58dc510037d5efa622f9bec8519bc1beab13c9f251e97e657a9d4ed)

## [N-07] NEED FUZZING TEST
We recommend the use of fuzzing tests, especially in finance oriented protocols, due to the complexity and risk involved in handling large amounts of money in these smart contracts. Finance oriented contracts are critical in terms of security and accuracy, as any errors or vulnerabilities could be exploited by malicious attackers to steal funds or cause significant damage. 

Fuzzing tests are an important tool for identifying possible vulnerabilities in the code through the automatic and random generation of input data in the contract, which can help avoid costly errors in production.

### RECOMMENDATION 
Take advantage of testing with Foundry and implement [fuzzing](https://book.getfoundry.sh/forge/fuzz-testing) on your functions.

## [N-08] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()
Since version 0.8.4 for appending bytes, [bytes.concat()](https://docs.soliditylang.org/en/v0.8.19/types.html#bytes-concat) can be used instead of abi.encodePacked().

### PROOF OF CONCEPT
```solidity
main/src/PrivatePoolMetadata.sol
// @audit Use byte.concat() instead of abi.encodePacked() 

19: bytes memory metadata = abi.encodePacked
30: return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));
39: bytes memory _attributes = abi.encodePacked
62: _svg = abi.encodePacked
81: _svg = abi.encodePacked
97:_svg = abi.encodePacked
114: return string(abi.encodePacked(...
```

## [N-9] LINES ARE TOO LONG
Traditionally, source code lines have been limited to 80 characters, but with today's larger screens, it is reasonable to exceed this limit in certain cases. However, if the files are likely to be hosted on GitHub, it is important to note that GitHub will display a scrollbar for lines over 164 characters in length. Therefore, to ensure optimal readability, lines should be split when they reach this length.

(why-is-80-characters-the-standard-limit-for-code-width)[https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width]
- Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

### PROOF OF CONCEPT
```solidity
main/src/PrivatePoolMetadata.sol.attributes

// @audit Keep lines short to maintain a clean project and adhere to Solidity style standards.
    47: trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))

    main/src/PrivatePoolMetadata.sol.svg
    103: "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),

    main/src/PrivatePool.sol
    58: event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

    63: event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
```
### RECOMMENDATION
Multiline output parameters and return statements should follow the same style recommended for wrapping long lines found in the Maximum Line Length section.
We recommend following your own best practice patterns like:

```solidity
main/src/PrivatePool.sol

 function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {...}
```

## [N-10] SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE
Short-circuiting is a contract development model in Solidity that uses OR/AND logic to sequence different operations with varying costs. The idea is to place low-cost operations at the front and high-cost operations at the back, so that if the low-cost operation at the front is viable, the subsequent higher-cost Ethereum virtual machine operation can be skipped (short-circuited). This approach not only improves contract efficiency but can also reduce transaction costs by minimizing gas consumption.
```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```
### PROOF OF CONCEPT
```solidity
blob/main/src/Factory.sol

// @audit Sort operations
    87: if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

    main/src/EthRouter.sol
    101: if (block.timestamp > deadline && deadline != 0) {
    154: if (block.timestamp > deadline && deadline != 0) {
    228: if (block.timestamp > deadline && deadline != 0) {
    234: if (price > maxPrice || price < minPrice) {
    256: if (block.timestamp > deadline && deadline != 0) {

    main/src/PrivatePool.sol
    225: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
    277: if (royaltyFee > 0 && recipient != address(0)) {
    244: if (royaltyFee > 0 && recipient != address(0)) {
    397: if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
    489: if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
    635: if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
```

## [N-11] CONTRACT DOES NOT FOLLOW THE SOLIDITY STYLE GUIDE'S SUGGESTED LAYOUT ORDERING
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be: 
- 1. Type declarations 
- 2. State variables,
- 3. Events 
- 4. Modifiers 
- 5. Functions 
but the contract(s) below do not follow this ordering.

- [Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol)
- [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

## [N-12] ASSEMBLY CODES SPECIFIC - SHOULD HAVE COMMENTS
Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

### PROOF OF CONCEPT
```solidity
main/src/PrivatePool.sol
469: assembly {
                let returnData_size := mload(returnData)
                revert(add(32, returnData), returnData_size)
            }       
```

## [R-01] FUNCTION NAMING SUGGESTIONS
Proper use of _ as a function name prefix and a common pattern is to prefix internal and private function names with _. This pattern is correctly applied in all contracts, however there are some inconsistencies in just these contracts.

### PROOF OF CONCEPT
```solidity
main/src/PrivatePoolMetadata.sol

// @audit Refactorize
    112: function trait(string memory traitType, string memory value) internal pure returns (string memory) {
            // forgefmt: disable-next-item
            return string(
                abi.encodePacked(
                    '{ "trait_type": "', traitType, '",', '"value": "', value, '" }'
                )
            );
        }
```
### RECOMMENDATION
```diff
-112: function trait(string memory traitType, string memory value) internal pure returns (string memory)
+112: function _trait(string memory traitType, string memory value) internal pure returns (string memory) 
```

## [R-02] SHORTHAND WAY TO WRITE IF/ELSE STATEMENT
The regular if-else structure can be simplified through an abbreviated version. This technique not only improves code readability but also helps reduce the total number of lines of code (SLOC).

### PROOF OF CONCEPT
```solidity
main/src/Factory.sol

// @audit Refactorize
    110: if (_baseToken == address(0)) {
                // transfer eth into the pool if base token is ETH
                address(privatePool).safeTransferETH(baseTokenAmount);
            } else {
                // deposit the base tokens from the caller into the pool
                ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);
        }

    149: if (token == address(0)) {
                msg.sender.safeTransferETH(amount);
            } else {
                ERC20(token).transfer(msg.sender, amount);
        }

    main/src/PrivatePool.sol
    278: if (baseToken != address(0)) {
                            ERC20(baseToken).safeTransfer(recipient, royaltyFee);
                        } else {
                            recipient.safeTransferETH(royaltyFee);
                        }

    345: if (baseToken != address(0)) {
                            ERC20(baseToken).safeTransfer(recipient, royaltyFee);
                        } else {
                            recipient.safeTransferETH(royaltyFee);
                        }

    467: if (returnData.length > 0) {
                // solhint-disable-next-line no-inline-assembly
                assembly {
                    let returnData_size := mload(returnData)
                    revert(add(32, returnData), returnData_size)
                }
            } else {
                revert();
            }

    522: if (token == address(0)) {
                // transfer the ETH to the caller
                msg.sender.safeTransferETH(tokenAmount);
            } else {
                // transfer the tokens to the caller
                ERC20(token).transfer(msg.sender, tokenAmount);
            }
```

## [R-03] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO)
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/catellaTech/8539f7345b17f0929a41598e7e00e3c2) in Solidity. The same applies to the subtraction operation.

### PROOF OF CONCEPT
```solidity
main/src/PrivatePool.sol

// @audit Refactorize

    230: virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
    231: virtualNftReserves -= uint128(weightSum);
    247: royaltyFeeAmount += royaltyFee;
    252: netInputAmount += royaltyFeeAmount;
    323: virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
    324: virtualNftReserves += uint128(weightSum);
    341: royaltyFeeAmount += royaltyFee;
    355: netOutputAmount -= royaltyFeeAmount;
    678: sum += tokenWeights[i];
```
### RECOMENDATION
```diff
main/src/PrivatePool.sol#L247
- 247: royaltyFeeAmount += royaltyFee;
+ 247: royaltyFeeAmount = royaltyFeeAmount + royaltyFee;
```
