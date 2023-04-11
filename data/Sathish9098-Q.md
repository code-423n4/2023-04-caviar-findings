## LOW FINDINGS 

|  Low Count | Issues | Instances |
| -------- | --------| -------- |
| [L-1]  | Lack of address(0) check when assigning address to state variables  | 4  |
| [L-2]  | A single point of failure | 8  |
| [L-3]  | Front running attacks by the onlyOwner | 8  |
| [L-4]  | Prevent division by 0 | 5  |
| [L-5]  | Sanity/Threshold/Limit Checks | 10  |
| [L-6]  | Consider using OpenZeppelin’s SafeCast library to prevent unexpected errors  | 3  |
| [L-7]  | Function may run out of gas | 7  |
| [L-8]  | Gas griefing/theft is possible on unsafe external call | 1  |
| [L-9]  | Loss of precision due to rounding | 11  |
| [L-10]  | Use latest @openzeppelin/merkle-tree version  | 1  |
| [L-11]  | abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256() | 7  |

# NON CRITICAL FINDINGS

| NC Count | Issues | Instances |
| -------- | --------| -------- |
| [NC-1]   | Add a timelock to critical functions | 8  |
| [NC-2]   | No same value control  | 8  |
| [NC-3]   | Critical changes should use two-step procedure | 8  |
| [NC-4]   | Events that mark critical parameter changes should contain both the old and the new value | 8 |
| [NC-5]   | NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS | -  |
| [NC-6]   | NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING | 11 |
| [NC-7]   | Mark visibility of initialize(…) functions as external | 1  |
| [NC-8]   | Contract layout and order of functions | 8  |
| [NC-9]   | Pragma float | -  |
| [NC-10]   | immutable should be uppercase  | 3  |
| [NC-11]   | Public functions not called by contract can declare as external  | 11  |
| [NC-12]   | Include return parameters in NatSpec comments | 6  |
| [NC-13]   | Use of bytes.concat() instead of abi.encodePacked() | 7  |
| [NC-14]   | For functions, follow Solidity standard naming conventions (internal function style rule) | 1 |
| [NC-15]   | Typos | 1  |
| [NC-16]   | NatSpec is incomplete | 3  |
| [NC-17]   | Missing NATSPEC | 5  |
| [NC-18]   | Lines are too long | 4  |
| [NC-19]   | Non-library/interface files should use fixed compiler versions, not floating ones | 1 |
| [NC-20]   | Event is missing indexed fields | 15  |
| [NC-21]   | Duplicated require()/revert() checks should be refactored to a modifier or function | 12 |
| [NC-22]   | Imports can be grouped together | 1  |
| [NC-23]   | Approve() function return value not checked | 1  |
| [NC-24]   | Assembly Codes Specific – Should Have Comments | 1  |

##

## [L-1] Lack of address(0) check when assigning address to state variables 

> Instances(4) 

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

   constructor(address _royaltyRegistry) {
        royaltyRegistry = _royaltyRegistry;
    }
```
[EthRouter.sol#L90-L92](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L90-L92)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

143: constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
144:        factory = payable(_factory);
145:        royaltyRegistry = _royaltyRegistry;
146:        stolenNftOracle = _stolenNftOracle;
147:    }

```
[PrivatePool.sol#L143-L147](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L143-L147)

##

## [L-2] A single point of failure

> Instances(8) 

### Impact
The onlyOwner role has a single point of failure and onlyOwner can use critical a few functions.

Even if protocol admins/developers are not malicious there is still a chance for Owner keys to be stolen. In such a case, the attacker can cause serious damage to the project due to important functions. In such a case, users who have invested in project will suffer high financial losses.

onlyOwner functions;

```solidity
FILE: 2023-04-caviar/src/Factory.sol

129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

```
[Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129),[Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135),[Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141),[Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L148)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

538:  function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
550:  function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
562:  function setFeeRate(uint16 newFeeRate) public onlyOwner {
576:  function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
587:  function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```
[PrivatePool.sol#L538](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538),[PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L550),[PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L562),[PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L576),[PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L587)

### Recommended Mitigation Steps
Add a time lock to critical functions. Admin-only functions that change critical parameters should emit events and have timelocks.

Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services.

Also detail them in documentation and NatSpec comments.

##

## [L-3] Front running attacks by the onlyOwner

> Instances(8) 

```solidity
FILE: 2023-04-caviar/src/Factory.sol

129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

```
[Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129),[Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135),[Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141),[Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L148)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

538:  function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
550:  function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
562:  function setFeeRate(uint16 newFeeRate) public onlyOwner {
576:  function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
587:  function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```
[PrivatePool.sol#L538](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538),[PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L550),[PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L562),[PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L576),[PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L587)



### Recommended Mitigation Steps
Use a timelock to avoid instant changes of the parameters.


## [L-4] Prevent division by 0

> Instances(5) 

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

```solidity
FILE : 2023-04-caviar/src/EthRouter.sol

115: uint256 salePrice = inputAmount / buys[i].tokenIds.length;
182: uint256 salePrice = outputAmount / sells[i].tokenIds.length;

```
[EthRouter.sol#L115](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L115)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

236:  uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
335:  uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;
745:  return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;

```
[PrivatePool.sol#L236](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L236),[PrivatePool.sol#L335](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L335)



## [L-5] Sanity/Threshold/Limit Checks

> Instances(10) 

Devoid of sanity/threshold/limit checks, critical parameters can be configured to invalid values, causing a variety of issues and breaking expected interactions within/between contracts. Consider adding proper uint256 validation as well as zero address checks for critical changes. A worst case scenario would render the contract needing to be re-deployed in the event of human/accidental errors that involve value assignments to immutable variables. If the validation procedure is unclear or too complex to implement on-chain, document the potential issues that could produce invalid values

```solidity
FILE: 2023-04-caviar/src/Factory.sol

_privatePoolImplementation address(0) not checked for critical assignments 

function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }

_privatePoolImplementation address(0) not checked for critical assignments 

function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }

_protocolFeeRate value not checked before assigning . There is possibility of 0 values 

function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }

The amount not checked 

function withdraw(address token, uint256 amount) public onlyOwner {
        if (token == address(0)) {
            msg.sender.safeTransferETH(amount);
        } else {
            ERC20(token).transfer(msg.sender, amount);
        }

        emit Withdraw(token, amount);
    }
```
[Factory.sol#L129-L156](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L156)

[PrivatePool.sol#L538-L615](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L615)



##

## [L-6] Consider using OpenZeppelin’s SafeCast library to prevent unexpected errors 

> Instances(3)

Using OpenZeppelin's SafeCast library can help prevent unexpected errors when casting variables from one data type to another. SafeCast provides a set of casting functions that perform type conversions in a safe and secure way, by checking for possible overflow or underflow conditions and throwing an error if any such condition is detected

```solidity
FILE: 2023-04-caviar/src/Factory.sol

95: _safeMint(msg.sender, uint256(uint160(address(privatePool))));

```
[Factory.sol#L95](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L95)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

230: virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
231: virtualNftReserves -= uint128(weightSum);

```

Recommended Mitigation Steps:
Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from uint256.

##

## [L-7] Function may run out of gas

> Instances(7) 

The for loop in the Solidity code you provided, for (uint256 i = 0; i < tokenIds.length; i++) { ... }, has the potential to consume a lot of gas, especially if the tokenIds array is large. This is because each iteration of the loop requires the execution of the code inside the loop, which can result in a significant amount of computational work.

If the gas limit for the transaction executing this loop is not set high enough to cover the gas cost of each iteration of the loop, the transaction may run out of gas and fail. When a transaction runs out of gas, all state changes made up to that point are reverted and any ether sent with the transaction is lost

```solidity
FILE: 2023-04-caviar/src/Factory.sol

 for (uint256 i = 0; i < tokenIds.length; i++) {
            ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
        }

```
[Factory.sol#L119-L121](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L119-L121)

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
                // transfer the NFT to the caller
                ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
            }


for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
                ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
            }


for (uint256 i = 0; i < tokenIds.length; i++) {
            ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
        }

for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
                ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);
            }


```
[EthRouter.sol#L134-L138](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L134-L138),[EthRouter.sol#L161-L163](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L161-L163),[EthRouter.sol#L239-L241](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L239-L241),[EthRouter.sol#L284-L286](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L284-L286)

```solidity 

for (uint256 i = 0; i < inputTokenIds.length; i++) {
            ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
        }

        // transfer the output nfts to the caller
        for (uint256 i = 0; i < outputTokenIds.length; i++) {
            ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
        }

```
[PrivatePool.sol#L441-L448](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L441-L448)

##

## [L-8] Gas griefing/theft is possible on unsafe external call

> Instances(1) 

return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

461: (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

```
[PrivatePool.sol#L461](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L461)

##

## [L-9] Loss of precision due to rounding

> Instances(11)

Add scalars so roundings are negligible.

```solidity
FILE : 2023-04-caviar/src/EthRouter.sol

115:  uint256 salePrice = inputAmount / buys[i].tokenIds.length;
182:  uint256 salePrice = outputAmount / sells[i].tokenIds.length;

```
[EthRouter.sol#L115](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L115)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

236: uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
335: uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;
703: protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() / 10_000;
704: feeAmount = inputAmount * feeRate / 10_000;
719: uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);
721: protocolFeeAmount = outputAmount * Factory(factory).protocolFeeRate() / 10_000;
722: feeAmount = outputAmount * feeRate / 10_000;
737: protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;
745: return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;

```
[PrivatePool.sol#L236](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L236)

##

## [L-10] Use latest @openzeppelin/merkle-tree version 

Latest version is 1.0.4 

```
FILE: package-lock.json

12: "@openzeppelin/merkle-tree": "^1.0.2",

```
##

## [L-11] abi.encodePacked() should not be used with dynamic types when passing the result to a hash function such as keccak256()

Use abi.encode() instead which will pad items to 32 bytes, which will [prevent hash collisions](https://docs.soliditylang.org/en/v0.8.13/abi-spec.html#non-standard-packed-mode) (e.g. abi.encodePacked(0x123,0x456) => 0x123456 => abi.encodePacked(0x1,0x23456), but abi.encode(0x123,0x456) => 0x0...1230...456). “Unless there is a compelling reason, abi.encode should be preferred”. If there is only one argument to abi.encodePacked() it can often be cast to bytes() or bytes32() instead.

If all arguments are strings and or bytes, bytes.concat() should be used [instead](https://ethereum.stackexchange.com/questions/30912/how-to-compare-strings-in-solidity#answer-82739).

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L19-L28
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L30
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L39-L48
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L62-L76
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L81-L92
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L97-L106
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L115-L117

##

##



##

## NON CRITICAL FINDINGS

## [NC-1] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to.

```solidity
FILE: 2023-04-caviar/src/Factory.sol

function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }

function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }

function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }


```
[Factory.sol#L129-L143](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L143)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
        // set the virtual base token reserves and virtual nft reserves
        virtualBaseTokenReserves = newVirtualBaseTokenReserves;
        virtualNftReserves = newVirtualNftReserves;

        // emit the set virtual reserves event
        emit SetVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);
    }

    /// @notice Sets the merkle root. Can only be called by the owner of the pool. The merkle root is used to validate
    /// the NFT weights.
    /// @param newMerkleRoot The new merkle root.
    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
        // set the merkle root
        merkleRoot = newMerkleRoot;

        // emit the set merkle root event
        emit SetMerkleRoot(newMerkleRoot);
    }

    /// @notice Sets the fee rate. Can only be called by the owner of the pool. The fee rate is used to calculate the
    /// fee amount when swapping or changing NFTs. The fee rate is in basis points (1/100th of a percent). For example,
    /// 10_000 == 100%, 200 == 2%, 1 == 0.01%.
    /// @param newFeeRate The new fee rate (in basis points)
    function setFeeRate(uint16 newFeeRate) public onlyOwner {
        // check that the fee rate is less than 50%
        if (newFeeRate > 5_000) revert FeeRateTooHigh();

        // set the fee rate
        feeRate = newFeeRate;

        // emit the set fee rate event
        emit SetFeeRate(newFeeRate);
    }

    /// @notice Sets the whether or not to use the stolen NFT oracle. Can only be called by the owner of the pool. The
    /// stolen NFT oracle is used to check if an NFT is stolen.
    /// @param newUseStolenNftOracle The new use stolen NFT oracle flag.
    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
        // set the use stolen NFT oracle flag
        useStolenNftOracle = newUseStolenNftOracle;

        // emit the set use stolen NFT oracle event
        emit SetUseStolenNftOracle(newUseStolenNftOracle);
    }

    /// @notice Sets the pay royalties flag. Can only be called by the owner of the pool. If royalties are enabled then
    /// the pool will pay royalties when buying or selling NFTs.
    /// @param newPayRoyalties The new pay royalties flag.
    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
        // set the pay royalties flag
        payRoyalties = newPayRoyalties;

        // emit the set pay royalties event
        emit SetPayRoyalties(newPayRoyalties);
    }

```

[PrivatePool.sol#L538-L593](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L593)

##

## [NC-2] No same value control 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L143
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L593


##

## [NC-3] Critical changes should use two-step procedure

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two-step procedure on the critical functions.

Consider adding a two-steps pattern on critical changes to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L143
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L593

##

## [NC-4] Events that mark critical parameter changes should contain both the old and the new value

Emitting old and new values when critical changes are made can help you improve the reliability, accuracy, and maintainability of your systems

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L143
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L593

##

## [NC-5] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

### Recommendation
NatSpec comments should be increased in Contracts

##

## [NC-6] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

Consider changing the variable to be an unnamed one

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211-L214
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L291-L306
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L375-L393
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L694-L697
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L713-L716
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L731
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L778-L781
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L71-L84
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L168
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L301-L304
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L385-L393

##

## [NC-7] Mark visibility of initialize(…) functions as external

Description
External instead of public would give more the sense of the initialize(…) functions to behave like a constructor (only called on deployment, so should only be called externally).

Security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

It might cost a bit less gas to use external over public.

It is possible to override a function from external to public (= “opening it up”) ✅
but it is not possible to override a function from public to external (= “narrow it down”). ❌

For above reasons you can change initialize(…) to external

https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

function initialize(
        address _baseToken,
        address _nft,
        uint128 _virtualBaseTokenReserves,
        uint128 _virtualNftReserves,
        uint56 _changeFee,
        uint16 _feeRate,
        bytes32 _merkleRoot,
        bool _useStolenNftOracle,
        bool _payRoyalties
    ) public {

```
[PrivatePool.sol#L157-L167](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L157-L167)
##

## [NC-8] Contract layout and order of functions

The Solidity style guide recommends declaring state variables before all functions. 

(https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout)

CONTEXT
ALL CONTRACT

### Layout contract elements in the following order:

- Pragma statements

- Import statements

Interfaces

- Libraries

- Contracts

### Inside each contract, library or interface, use the following order:

- Type declarations

- State variables

- Events

- Modifiers

- Functions

### Recommendations 

All contracts should follow the solidity style guide 

##

## [NC-9] Pragma float

### CONTEXT
ALL CONTRACTS

### Recommendation
Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package

##

## [NC-10] immutable should be uppercase 

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

119: address public immutable stolenNftOracle;
122: address payable public immutable factory;
125: address public immutable royaltyRegistry;

```
##
## [NC-11] Public functions not called by contract can declare as external 

The following functions could be set external to improve code quality

```solidity
FILE : 2023-04-caviar/src/PrivatePoolMetadata.sol

17: function tokenURI(uint256 tokenId) public view returns (string memory) {
35: function attributes(uint256 tokenId) public view returns (string memory) {
55: function svg(uint256 tokenId) public view returns (bytes memory) {

```
[PrivatePoolMetadata.sol#L35](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L35)

```solidity
FILE: 2023-04-caviar/src/Factory.sol

function create(
        address _baseToken,
        address _nft,
        uint128 _virtualBaseTokenReserves,
        uint128 _virtualNftReserves,
        uint56 _changeFee,
        uint16 _feeRate,
        bytes32 _merkleRoot,
        bool _useStolenNftOracle,
        bool _payRoyalties,
        bytes32 _salt,
        uint256[] memory tokenIds, // put in memory to avoid stack too deep error
        uint256 baseTokenAmount
    ) public payable returns (PrivatePool privatePool) {

135:  function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148: function withdraw(address token, uint256 amount) public onlyOwner {

129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

168: function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

```

[Factory.sol#L71-L84](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L71-L84),[Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135),[Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141),[Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L148),[Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129),[Factory.sol#L168](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L168)

```solidity
FILE: 2023-04-caviar/src/PrivatePoolMetadata.sol

17: function tokenURI(uint256 tokenId) public view returns (string memory) {


```
[PrivatePoolMetadata.sol#L17](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L17)

##

## [Nc-12] Include return parameters in NatSpec comments

> Some functions return parameters is missing

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L33-L35
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L15-L17
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L53-L55
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L375-L393



##

## [NC-13] Use of bytes.concat() instead of abi.encodePacked()

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L19-L28
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L30
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L39-L48
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L62-L76
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L81-L92
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L97-L106
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L115-L117

##

## [NC-14] For functions, follow Solidity standard naming conventions (internal function style rule)

### Description
The above codes don’t follow Solidity’s standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L112)

##

## [NC-15] Typos

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

/// @audit aount => amount
251: // add the royalty fee amount to the net input aount

```

##

## [NC-16] NatSpec is incomplete

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

/// @audit Missing: '@return' of protocolFeeAmount

    /// @param proof The merkle proof for the weights of each NFT to buy.
    /// @return netInputAmount The amount of base tokens spent inclusive of fees.
    /// @return feeAmount The amount of base tokens spent on fees.
    function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata 
    proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)


/// @audit Missing: '@return' of protocolFeeAmount
/// @param stolenNftProofs The proofs that show each NFT is not stolen.
    /// @return netOutputAmount The amount of base tokens received inclusive of fees.
    /// @return feeAmount The amount of base tokens to pay in fees.
    function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {


/// @audit Missing: '@return' of protocolFeeAmount
/// @notice Returns the required input of buying a given amount of NFTs inclusive of the fee which is dependent on
    /// the currently set fee rate.
    /// @param outputAmount The amount of NFTs to buy multiplied by 1e18.
    /// @return netInputAmount The required input amount of base tokens inclusive of the fee.
    /// @return feeAmount The fee amount.
    function buyQuote(uint256 outputAmount)
        public
        view
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

```

##

## [NC-17] Missing NATSPEC

NATSPEC Missing for structs 

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

struct Buy {
        address payable pool;
        address nft;
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        uint256 baseTokenAmount;
        bool isPublicPool;
    }

    struct Sell {
        address payable pool;
        address nft;
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        IStolenNftOracle.Message[] stolenNftProofs;
        bool isPublicPool;
        bytes32[][] publicPoolProofs;
    }

    struct Change {
        address payable pool;
        address nft;
        uint256[] inputTokenIds;
        uint256[] inputTokenWeights;
        PrivatePool.MerkleMultiProof inputProof;
        IStolenNftOracle.Message[] stolenNftProofs;
        uint256[] outputTokenIds;
        uint256[] outputTokenWeights;
        PrivatePool.MerkleMultiProof outputProof;
    }

86: address public immutable royaltyRegistry;

90: constructor(address _royaltyRegistry) {
91:        royaltyRegistry = _royaltyRegistry;
92:    }

```

##

## [NC-18] Lines are too long

Usually lines in source code are limited to [80](https://softwareengineering.stackexchange.com/questions/148677/why-is-80-characters-the-standard-limit-for-code-width) characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over [164](https://github.com/aizatto/character-length) characters, the lines below should be split when they reach that length

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

58: event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);
59: event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
60: event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);

63: event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);

```

##

## [NC-19] Non-library/interface files should use fixed compiler versions, not floating ones

```solidity
FILE: 2023-04-caviar/src/interfaces/IStolenNftOracle.sol

2: pragma solidity ^0.8.19;

```

##

## [NC-20] Event is missing indexed fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it’s not necessarily best to index the maximum allowed per event (threefields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed

```solidity
FILE : 2023-04-caviar/src/Factory.sol

41: event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);
```

[Factory.sol#L41](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L41)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

58: event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);
59: event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
60: event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
61: event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);
62: event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);
63: event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
64: event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);
65: event SetMerkleRoot(bytes32 merkleRoot);
66: event SetFeeRate(uint16 feeRate);
67: event SetUseStolenNftOracle(bool useStolenNftOracle);
68: event SetPayRoyalties(bool payRoyalties);

```
[PrivatePool.sol#L58-L68](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L58-L68)

##

## [NC-21] Duplicated require()/revert() checks should be refactored to a modifier or function

The compiler will inline the function, which will avoid JUMP instructions usually associated with functions

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

101: if (block.timestamp > deadline && deadline != 0) {
154: if (block.timestamp > deadline && deadline != 0) {
228: if (block.timestamp > deadline && deadline != 0) {
256: if (block.timestamp > deadline && deadline != 0) {

```
[EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L101),[EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L154),[EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L228),[EthRouter.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L256)

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

254: if (baseToken != address(0)) {
278: if (baseToken != address(0)) {
345: if (baseToken != address(0)) {
421: if (baseToken != address(0)) {
500: if (baseToken != address(0)) {
651:  if (baseToken != address(0)) 

277: if (royaltyFee > 0 && recipient != address(0)) {
344: if (royaltyFee > 0 && recipient != address(0)) {

```
[PrivatePool.sol#L254](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L254)

##

## [NC-22] Imports can be grouped together

Consider importing like this format -  solmate,solady,openzeppelin,royalty-registry-solidity,interfaces,Factory

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

 32: import {IERC2981} from "openzeppelin/interfaces/IERC2981.sol";
+34: import {IERC3156FlashBorrower} from "openzeppelin/interfaces/IERC3156FlashLender.sol";
 33: import {IRoyaltyRegistry} from "royalty-registry-solidity/IRoyaltyRegistry.sol";
-34: import {IERC3156FlashBorrower} from "openzeppelin/interfaces/IERC3156FlashLender.sol";

```

(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L27-L37)

##

## [NC-23] Approve() function return value not checked 

```solidity
FILE: 2023-04-caviar/src/EthRouter.sol

166: ERC721(sells[i].nft).setApprovalForAll(sells[i].pool, true);

```
[EthRouter.sol#L166](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L166)

##

## [NC-24] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

469: assembly {

```
[PrivatePool.sol#L469](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L469)

##









NC-1	Constants should be defined rather than using magic numbers	1
L-1	Empty Function Body - Consider commenting why	4
L-2	Initializers could be front-run	2
L-3	Unsafe ERC20 operation(s)	5
