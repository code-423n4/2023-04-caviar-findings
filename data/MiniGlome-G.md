## Gas Optimizations
| |Issue|Instances|
|-|:-|:-:|
| [GAS-01] | Structs can be packed into fewer storage slots | 2 |
| [GAS-02] | Functions guaranteed to revert when called by normal users can be marked `payable` | 11 | 
| [GAS-03] | Setting the `constructor` to `payable` | 3 | 
| [GAS-04] | Usage of uint/int smaller than 32 bytes | 9 | 
| [GAS-05] | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 4 | 

### [GAS-01] Structs can be packed into fewer storage slots
Structs can be packed in slot of 32 bytes like state variables to reduce storage usage.
See more: https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html

*Instances (2)*:
```solidity
File: src/EthRouter.sol
48:    struct Buy {
        address payable pool;
        address nft;
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        uint256 baseTokenAmount;
        bool isPublicPool;
    }

58:    struct Sell {
        address payable pool;
        address nft;
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        IStolenNftOracle.Message[] stolenNftProofs;
        bool isPublicPool;
        bytes32[][] publicPoolProofs;
    }

```
Can be packed like this:
```solidity
    struct Buy {
        address payable pool;
        address nft;
        bool isPublicPool;
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        uint256 baseTokenAmount;        
    }

    struct Sell {
        address payable pool;
        address nft;
        bool isPublicPool;
        uint256[] tokenIds;
        uint256[] tokenWeights;
        PrivatePool.MerkleMultiProof proof;
        IStolenNftOracle.Message[] stolenNftProofs;        
        bytes32[][] publicPoolProofs;
    }

```

### [GAS-02] Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are `CALLVALUE`(2),`DUP1`(3),`ISZERO`(3),`PUSH2`(3),`JUMPI`(10),`PUSH1`(3),`DUP1`(3),`REVERT`(0),`JUMPDEST`(1),`POP`(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

*Instances (11)*:
```solidity
File: src/Factory.sol
129:    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:    function withdraw(address token, uint256 amount) public onlyOwner {

```

```solidity
File: src/PrivatePool.sol
459:    function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

514:    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538:    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550:    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562:    function setFeeRate(uint16 newFeeRate) public onlyOwner {

576:    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587:    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```

### [GAS-03] Setting the `constructor` to `payable`
Saves ~13 gas per instance

*Instances (3)*:
```solidity
File: src/EthRouter.sol
90:    constructor(address _royaltyRegistry) {

```

```solidity
File: src/Factory.sol
53:    constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}

```

```solidity
File: src/PrivatePool.sol
143:    constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {

```

### [GAS-04] Usage of uint/int smaller than 32 bytes
When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html<br>Use a larger size then downcast where needed.

*Instances (9)*:
```solidity
File: src/Factory.sol
51:    uint16 public protocolFeeRate;

77:        uint16 _feeRate,

141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

```

```solidity
File: src/PrivatePool.sol
58:    event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);

66:    event SetFeeRate(uint16 feeRate);

91:    uint16 public feeRate;

163:        uint16 _feeRate,

562:    function setFeeRate(uint16 newFeeRate) public onlyOwner {

606:        uint16 newFeeRate,

```

### [GAS-05] `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables
Using the addition operator instead of plus-equals saves **[113 gas](https://gist.github.com/MiniGlome/f462d69a30f68c89175b0ce24ce37cae)**
Same for `-=`, `*=` and `/=`.

*Instances (4)*:
```solidity
File: src/PrivatePool.sol
230:        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231:        virtualNftReserves -= uint128(weightSum);

323:        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324:        virtualNftReserves += uint128(weightSum);

```

