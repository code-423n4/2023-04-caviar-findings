# Report
## Low Risk ##
### [L-1]: Unsafe casting may overflow
**Context:**

1. ```virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);``` [L230](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230) 
1. ```virtualNftReserves -= uint128(weightSum);``` [L231](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231) 
1. ```virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);``` [L323](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323) 
1. ```virtualNftReserves += uint128(weightSum);``` [L324](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324) 

**Description:**

While Solidity 0.8.x checks for overflows on arithmetic operations, it does not do so for casting.

**Recommendation:**

Use OpenZeppelin’s SafeCast library to prevent unexpected overflows.

### [L-2]: transferOwnership should be two step process
**Context:**

1. ```import {Owned} from "solmate/auth/Owned.sol";``` [L26](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L26) check

**Description:**

"solmate/auth/Owned.sol" implements transferOwnership function. Lack of two-step procedure for transfer of ownership makes it error-prone. It’s possible that the owner mistakenly transfers ownership to the uncontrolled account and it will break all functions with onlyOwner() modifier.

**Recommendation:**

Implement a two step process for transfer of ownership. Owner call transferOwnership()  function where nominates an account. Nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This will confirm that the nominated EOA account is a valid and active account.

### [L-3]: Omissions in Events
**Context:**

1. ```event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);``` [L64](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L64) (the events should include the new value and old value where possible)
1. ```event SetMerkleRoot(bytes32 merkleRoot);``` [L65](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L65) (the events should include the new value and old value where possible)
1. ```event SetFeeRate(uint16 feeRate);``` [L66](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L66) (the events should include the new value and old value where possible)
1. ```event SetUseStolenNftOracle(bool useStolenNftOracle);``` [L67](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L67) (the events should include the new value and old value where possible)
1. ```event SetPayRoyalties(bool payRoyalties);``` [L68](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L68) (the events should include the new value and old value where possible)
1. ```emit Buy(tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);``` [L288](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L288) (virtualBaseTokenReserves changes too, new virtualBaseTokenReserves must be included in event)
1. ```emit Buy(tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);``` [L288](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L288) (virtualNftReserves changes too, new virtualNftReserves must be included in event)
1. ```emit Sell(tokenIds, tokenWeights, netOutputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);``` [L372](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L372) (virtualBaseTokenReserves changes too, new virtualBaseTokenReserves must be included in event)
1. ```emit Sell(tokenIds, tokenWeights, netOutputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);``` [L372](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L372) (virtualNftReserves changes too, new virtualNftReserves must be included in event)

**Description:**

Events are generally emitted when sensitive changes are made to the contracts. Some events are missing important parameters.

### [L-4]: Critical changes should use two-step procedure
**Context:**

1. ```function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {``` [L129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129) 
1. ```function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {``` [L135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135) 
1. ```function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {``` [L141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141) 
1. ```function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {``` [L538](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538) 
1. ```function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {``` [L550](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550) 
1. ```function setFeeRate(uint16 newFeeRate) public onlyOwner {``` [L562](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562) 
1. ```function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {``` [L576](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576) 
1. ```function setPayRoyalties(bool newPayRoyalties) public onlyOwner {``` [L587](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587) 

**Recommendation:**

The best practice is to use two-step procedure for critical changes to make them less error-prone. 

## Non-Critical Issues ##
### [N-1]: Wrong order of functions
**Context:**

1. ```function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data)``` [L623](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L623) (external function can not go after public function)

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) functions should be grouped according to their visibility and ordered:

+ constructor

+ receive function (if exists)

+ fallback function (if exists)

+ external

+ public

+ internal

+ private

Within a grouping, place the view and pure functions last.

**Recommendation:**

Put the functions in the correct order according to the documentation.

### [N-2]: Missing leading underscores
**Context:**

1. ```function trait(string memory traitType, string memory value) internal pure returns (string memory) {``` [L112](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112) 

**Description:**

Internal and private functions, state variables, constants, and immutables should starting with an underscore.

### [N-3]: Recommendations for long function declarations
**Context:**

1. ```) public payable returns (PrivatePool privatePool) {``` [L84](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L84) (opening bracket should be placed on their own line)
1. ```) public payable {``` [L226](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L226) (opening bracket should be placed on their own line)
1. ```) public {``` [L167](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L167) (The closing parenthesis and opening bracket should be placed on their own line as)
1. ```) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {``` [L306](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L306) (opening bracket should be placed on their own line)
1. ```) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {``` [L393](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L393) (The closing parenthesis and opening bracket should be placed on their own line as)
1. ```) public {``` [L609](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L609) (opening bracket should be placed on their own line)
1. ```) public view returns (uint256) {``` [L665](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L665) (The closing parenthesis and opening bracket should be placed on their own line as)

**Description:**

https://docs.soliditylang.org/en/v0.8.19/style-guide.html#function-declaration
1. For long function declarations, it is recommended to drop each argument onto its own line at the same indentation level as the function body. The closing parenthesis and opening bracket should be placed on their own line as well at the same indentation level as the function declaration.
1. If a long function declaration has modifiers, then each modifier should be dropped to its own line.

### [N-4]: Include return parameters in NatSpec comments
**Context:**

1. ```function tokenURI(uint256 tokenId) public view returns (string memory) {``` [L17](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17) 
1. ```function attributes(uint256 tokenId) public view returns (string memory) {``` [L35](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L35) 
1. ```function svg(uint256 tokenId) public view returns (bytes memory) {``` [L55](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L55) 
1. ```returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)``` [L214](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L214) (protocolFeeAmount)
1. ```) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {``` [L393](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L393) (return feeAmount and protocolFeeAmount are missing)
1. ```returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount)``` [L716](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L716) (return protocolFeeAmount is missing)
1. ```) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {``` [L306](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L306) (return protocolFeeAmount)
1. ```returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)``` [L697](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L697) (return protocolFeeAmount is missing)

**Description:**

If Return parameters are declared, you must prefix them with /// @return.

Some code analysis programs do analysis by reading NatSpec details, if they can’t see the “@return” tag, they do incomplete analysis.

**Recommendation:**

Include return parameters in NatSpec comments.

### [N-5]: NatSpec is missing
**Context:**

1. ```function trait(string memory traitType, string memory value) internal pure returns (string memory) {``` [L112](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112) 
1. ```struct Buy {``` [L48](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L48) 
1. ```struct Sell {``` [L58](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L58) 
1. ```struct Change {``` [L69](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L69) 
1. ```constructor(address _royaltyRegistry) {``` [L90](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90) 

### [N-6]: Natspec is incomplete
**Context:**

1. ```bool _payRoyalties,``` [L80](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L80) (param _payRoyalties is missing)
1. ```function getRoyalty(address nft, uint256 tokenId, uint256 salePrice)``` [L301](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L301) (param nft is missing)
1. ```uint56 _changeFee,``` [L162](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L162) (param _changeFee is missing)
1. ```bool _payRoyalties``` [L166](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L166) (param _payRoyalties is missing)

### [N-7]: Line is too long
**Context:**

1. ```trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))``` [L47](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47) 
1. ```"Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),``` [L103](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103) 
1. ```receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");``` [L642](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L642) 

**Description:**

Maximum suggested line length is [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).
