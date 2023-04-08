##

##
## [L-1] Lack of address(0) check when assigning address to state variables 

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

## [L-1] A single point of failure

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

##

## [L-2] Front running attacks by the onlyOwner

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



## Recommended Mitigation Steps
Use a timelock to avoid instant changes of the parameters.


## [L-1] Prevent division by 0

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

```
[PrivatePool.sol#L236](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L236),[PrivatePool.sol#L335](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L335)



## [L-2] Sanity/Threshold/Limit Checks

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

```

##

## [L-4] Consider using OpenZeppelin’s SafeCast library to prevent unexpected errors 

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

## [L-5] Prevent division by 0

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

```solidity
FILE: 2023-04-caviar/src/PrivatePoolMetadata.sol

115: uint256 salePrice = inputAmount / buys[i].tokenIds.length;
182: uint256 salePrice = outputAmount / sells[i].tokenIds.length;
```
[EthRouter.sol#L115](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L115),[EthRouter.sol#L182](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L182),[]()

##

## [L-6] Function may run out of gas

The for loop in the Solidity code you provided, for (uint256 i = 0; i < tokenIds.length; i++) { ... }, has the potential to consume a lot of gas, especially if the tokenIds array is large. This is because each iteration of the loop requires the execution of the code inside the loop, which can result in a significant amount of computational work.

If the gas limit for the transaction executing this loop is not set high enough to cover the gas cost of each iteration of the loop, the transaction may run out of gas and fail. When a transaction runs out of gas, all state changes made up to that point are reverted and any ether sent with the transaction is lost

```solidity
FILE: 2023-04-caviar/src/Factory.sol

 for (uint256 i = 0; i < tokenIds.length; i++) {
            ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
        }

```
[Factory.sol#L119-L121](Factory.sol#L119-L121](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L119-L121)

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

## [L-7] Gas griefing/theft is possible on unsafe external call

return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

461: (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

```
[PrivatePool.sol#L461](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L461)

##








##

## NON CRITICAL FINDINGS



## [NC-4] Add a timelock to critical functions

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

The amount not checked 


```

[Factory.sol#L129-L143](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L143)

##

## [NC-5] No same value control 

```solidity


```


##

## [NC-6] Critical changes should use two-step procedure

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two-step procedure on the critical functions.

Consider adding a two-steps pattern on critical changes to avoid mistakenly transferring ownership of roles or critical functionalities to the wrong address

```solidity


```


##

## [NC-7] Emit both old and new values in critical changes 

Emitting old and new values when critical changes are made can help you improve the reliability, accuracy, and maintainability of your systems



##

## [NC-8] Missing NATSPEC

Consider adding NATSPEC on all public/external functions to improve documentation

##

## [NC-9] For functions, follow Solidity standard naming conventions (internal function style rule)

Description
The bellow codes don’t follow Solidity’s standard naming convention,

internal and private functions and variables : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

```solidity
```

##

## [NC-11] Considered good practice to include a descriptive error message

A good error message should be clear and concise, providing enough information to help the user or developer understand what went wrong and how to fix it. It should also be specific to the error that occurred, rather than a generic or vague message that doesn't provide any useful information

```solidity

```
##

## [NC-12] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation.
In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.
(https://docs.soliditylang.org/en/v0.8.15/natspec-format.html)

### Recommendation
NatSpec comments should be increased in Contracts

##

## [NC-13] NOT USING THE NAMED RETURN VARIABLES ANYWHERE IN THE FUNCTION IS CONFUSING

Consider changing the variable to be an unnamed one

(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211-L214)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L291-L306)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L375-L393)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L694-L697)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L713-L716)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L731)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L778-L781)


##

## [NC-14] Mark visibility of initialize(…) functions as external

Description
External instead of public would give more the sense of the initialize(…) functions to behave like a constructor (only called on deployment, so should only be called externally).

Security point of view, it might be safer so that it cannot be called internally by accident in the child contract.

It might cost a bit less gas to use external over public.

It is possible to override a function from external to public (= “opening it up”) ✅
but it is not possible to override a function from public to external (= “narrow it down”). ❌

For above reasons you can change initialize(…) to external

(https://github.com/OpenZeppelin/openzeppelin-contracts/issues/3750)

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

## [NC-15] Contract layout and order of functions

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

## [NC-16] Pragma float

### CONTEXT
ALL CONTRACTS

### Recommendation
Locking the pragma helps to ensure that contracts do not accidentally get deployed using an outdated compiler version.

Note that pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or a package

##

## [NC-17] immutable should be uppercase 

```solidity
FILE: 2023-04-caviar/src/PrivatePool.sol

119: address public immutable stolenNftOracle;
122: address payable public immutable factory;
125: address public immutable royaltyRegistry;

```
##
## [NC-18] Public functions not called by contract can declare as external 

The following functions could be set external to improve code quality:


```solidity
FILE : 2023-04-caviar/src/PrivatePoolMetadata.sol

17: function tokenURI(uint256 tokenId) public view returns (string memory) {
35: function attributes(uint256 tokenId) public view returns (string memory) {
55: function svg(uint256 tokenId) public view returns (bytes memory) {

```
[PrivatePoolMetadata.sol#L35](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L35)

##

## [Nc-19] Include return parameters in NatSpec comments

## Context

```
protocolFeeAmount return parameter is missing

```
[PrivatePool.sol#L202-L211](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L202-L211)

##

## [NC-20] Use of bytes.concat() instead of abi.encodePacked()

Rather than using abi.encodePacked for appending bytes, since version 0.8.4, bytes.concat() is enabled

Since version 0.8.4 for appending bytes, bytes.concat() can be used instead of abi.encodePacked(,)

(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L19-L28)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L30)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L39-L48)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L62-L76)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L81-L92)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L97-L106)
(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L115-L117)

##

## [NC-22] For Critical changes events should emit both old and new values 

(https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538-L615)





NC-1	Constants should be defined rather than using magic numbers	1
L-1	Empty Function Body - Consider commenting why	4
L-2	Initializers could be front-run	2
L-3	Unsafe ERC20 operation(s)	5
