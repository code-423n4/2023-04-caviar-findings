## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

```solidity
// define these state variables before all the events
82:    address public baseToken;
85:    address public nft;
88:    uint56 public changeFee;
91:    uint16 public feeRate;
94:    bool public initialized;
97:    bool public payRoyalties;
100:    bool public useStolenNftOracle;
104:    uint128 public virtualBaseTokenReserves;
112:    uint128 public virtualNftReserves;
116:    bytes32 public merkleRoot;
119:    address public immutable stolenNftOracle;
122:    address payable public immutable factory;
125:    address public immutable royaltyRegistry;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
// place this state variable before all the error declarations.
86:    address public immutable royaltyRegistry;
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), public, external, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

```solidity
// receive function should come right after constructor
134:    receive() external payable {}

// external function coming before public ones
623:    function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data)
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
// receive function should come after constructor
88:    receive() external payable {}
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol

```solidity
4:    interface IStolenNftOracle {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

```solidity
// consider indexing params when possible

58:    event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

59:    event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);

60:    event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);

61:    event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);

62:    event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);

63:    event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);

64:    event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);

65:    event SetMerkleRoot(bytes32 merkleRoot);

66:    event SetFeeRate(uint16 feeRate);

67:    event SetUseStolenNftOracle(bool useStolenNftOracle);

68:    event SetPayRoyalties(bool payRoyalties);

71:    error AlreadyInitialized();

72:    error Unauthorized();

73:    error InvalidEthAmount();

74:    error InvalidMerkleProof();

75:    error InsufficientInputWeight();

76:    error FeeRateTooHigh();

77:    error NotAvailableForFlashLoan();

78:    error FlashLoanFailed();

79:    error InvalidRoyaltyFee();

127:    modifier onlyOwner() virtual {

// not natSpec compliant
143:    constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {

// @param not documented
162:        uint56 _changeFee,

// @param not documented
166:        bool _payRoyalties

// @return missing
386:    function change(

// @params not defined
750:    function flashFee(address, uint256) public view returns (uint256) {

// @return not defined
755:    function flashFeeToken() public view returns (address) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
48:    struct Buy {

58:    struct Sell {

69:    struct Change {

81:    error DeadlinePassed();

82:    error OutputAmountTooSmall();

83:    error PriceOutOfRange();

84:    error InvalidRoyaltyFee();

90:    constructor(address _royaltyRegistry) {

// @param missing
301:    function getRoyalty(address nft, uint256 tokenId, uint256 salePrice)
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol

```solidity
// @return missing
17:    function tokenURI(uint256 tokenId) public view returns (string memory) {

// @return missing
35:    function attributes(uint256 tokenId) public view returns (string memory) {

// @return missing
55:    function svg(uint256 tokenId) public view returns (bytes memory) {

112:    function trait(string memory traitType, string memory value) internal pure returns (string memory) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
// @param _payRoyalties is not documented
80:        bool _payRoyalties,
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol

```solidity
// private and internal `functions` should preppend with `underline`
112:    function trait(string memory traitType, string memory value) internal pure returns (string memory) {
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol

```solidity
// lose the ^ for a safer code
2:  pragma solidity ^0.8.19;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

```solidity
// lose the ^ for a safer code
2:    pragma solidity ^0.8.19;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
// lose the ^ for a safer code
2:  pragma solidity ^0.8.19;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol

```solidity
// lose the ^ for a safer code
2:  pragma solidity ^0.8.19;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
// lose the ^ for a safer code
2:  pragma solidity ^0.8.19;
```

---


### Observations [6]

// some contracts are using `_param` style for functions arguments but others not, pick one and stick to it.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
// This contract is not emiting any events for important functions. Consider adding indexed events.
```
