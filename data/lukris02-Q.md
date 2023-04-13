# QA Report for Caviar Private Pools contest
## Overview
During the audit, 6 low and 5 non-critical issues were found.

№ | Title | Risk Rating  | Instance Count
--- | --- | --- | ---
L-1 | Use the two-step-transfer of ownership | Low | 1
L-2 | Use SafeCast Library | Low | 4
L-3 | Critical changes should use two-step procedure | Low | 8
L-4 | Add more information in the events | Low | 3
L-5 | Add a timelock to critical functions | Low | 3
L-6 | Limit protocol fee | Low | 1
NC-1 | Order of Functions | Non-Critical | 1
NC-2 | Maximum line length exceeded | Non-Critical | 3
NC-3 | Missing NatSpec | Non-Critical | 5
NC-4 | Natspec is incomplete | Non-Critical | 6
NC-5 | Missing leading underscores | Non-Critical | 1

## Low Risk Findings(6)
### L-1. Use the two-step-transfer of ownership
##### Description
If the owner accidentally transfers ownership to an incorrect address, protected functions may become permanently inaccessible.
##### Instances
- [```import {Owned} from "solmate/auth/Owned.sol";```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L26)

##### Recommendation
Consider using a two-step-transfer of ownership: the current owner would nominate a new owner, and to become the new owner, the nominated account would have to approve the change, so that the address is proven to be valid.
#
### L-2. Use SafeCast Library
##### Description
Downcasting from uint256/int256 in Solidity does not revert on overflow. This can easily result in undesired exploitation or bugs, since developers usually assume that overflows raise errors. SafeCast restores this intuition by reverting the transaction when such an operation overflows.
##### Instances
- [```virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230) 
- [```virtualNftReserves -= uint128(weightSum);```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231) 
- [```virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323) 
- [```virtualNftReserves += uint128(weightSum);```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324) 

##### Recommendation
It is better to use [safe casting library](https://docs.openzeppelin.com/contracts/3.x/api/utils#SafeCast).
#
### L-3. Critical changes should use two-step procedure
##### Description
Lack of two-step procedure for critical operations leaves them error-prone.
##### Instances
- [```function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129) 
- [```function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135) 
- [```function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141) 
- [```function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538) 
- [```function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550) 
- [```function setFeeRate(uint16 newFeeRate) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562) 
- [```function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576) 
- [```function setPayRoyalties(bool newPayRoyalties) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587) 

##### Recommendation
Consider adding two step procedure on the critical functions.
#
### L-4. Add more information in the events
##### Description
Some events are missing important information.
##### Instances
- [```event SetMerkleRoot(bytes32 merkleRoot);```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L65) (include previous merkleRoot)
- [```event SetFeeRate(uint16 feeRate);```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L66) (include previous feeRate)
- [```event SetPayRoyalties(bool payRoyalties);```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L68) (include previous payRoyalties)

#
### L-5. Add a timelock to critical functions
##### Description
Giving users time to react and adjust to critical changes in protocol provides more guarantees and increases the transparency of the protocol.
##### Instances
- [```function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135) 
- [```function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141) 
- [```function setFeeRate(uint16 newFeeRate) public onlyOwner {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562) 

##### Recommendation
Consider adding a timelock.
#
### L-6. Limit protocol fee
##### Description
The protocol will be more reliable if you limit the maximum protocol fee rate.
##### Instances
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L139-L143

##### Recommendation
Add a check that the fee rate being set do not exceed maxProtocolRate.
#
## Non-Critical Risk Findings(5)
### NC-1. Order of Functions
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-functions), ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.  
Functions should be grouped according to their visibility and ordered:
1) constructor
2) receive function (if exists)
3) fallback function (if exists)
4) external
5) public
6) internal
7) private
##### Instances
External function should not be placed between public function:
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L623

##### Recommendation
Reorder functions where possible.
#
### NC-2. Maximum line length exceeded
##### Description
According to [Style Guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#maximum-line-length), maximum suggested line length is 120 characters.  Longer lines make the code harder to read.
##### Instances
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L642

##### Recommendation
Make the lines shorter.
#
### NC-3. Missing NatSpec
##### Description
NatSpec is missing for 1 function and 3 structs in 2 contracts.
##### Instances
- [```function trait(string memory traitType, string memory value) internal pure returns (string memory) {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112) 
- [```struct Buy {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L48) 
- [```struct Sell {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L58) 
- [```struct Change {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L69) 
- [```constructor(address _royaltyRegistry) {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90) 

##### Recommendation
Add NatSpec for all functions.
#
### NC-4. Natspec is incomplete
##### Instances
- [```bool _payRoyalties,```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L80) (describe _payRoyalties)
- [```function getRoyalty(address nft, uint256 tokenId, uint256 salePrice)```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L301) (describe nft)
- [```uint56 _changeFee,```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L162) (describe _changeFee )
- [```bool _payRoyalties```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L166) (describe _payRoyalties)
- [```) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L306) (describe protocolFeeAmount)
- [```returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L697) (describe protocolFeeAmount)

##### Recommendation
Complement Natspec.
#
### NC-5. Missing leading underscores
##### Description
Internal functions should have a leading underscore.
##### Instances
- [```function trait(string memory traitType, string memory value) internal pure returns (string memory) {```](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112) 

##### Recommendation
Add a leading underscore.