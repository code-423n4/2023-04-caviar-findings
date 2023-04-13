# Gas Optimizations Report

Report Date: 2023-04-10 05:19:30
| |Issue|Instances|
|-|:-|:-:|
|[G-01]|State variables can be packed into fewer storage slots|2|
|[G-02]|x += y or x -= y costs more gas than x = x + y or x = x - y for state variables|5|
|[G-03]|Functions guaranteed to revert when called by normal users can be marked payable|11|
|[G-04]|++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops|19|
|[G-05]|public function that could be declared external|12|
|[G-06]| Optimize names to save gas|3|



## [G-01] State variables can be packed into fewer storage slots

### Impact
If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper.  


### Findings
Total:2

[src/Factory.sol#L45-L51](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol#L45-L51)
```solidity
45:    address public privatePoolImplementation;
46:    
47:        /// @notice Helper contract that constructs the private pool metadata svg and json for each pool NFT.
48:        address public privatePoolMetadata;
49:    
50:        /// @notice The protocol fee that is taken on each buy/sell/change. It's in basis points: 350 = 3.5%.
51:        uint16 public protocolFeeRate;
```
[src/PrivatePool.sol#L82-L100](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L82-L100)
```solidity
82:    address public baseToken;
83:    
84:        /// @notice The address of the nft.
85:        address public nft;
86:    
87:        /// @notice The change/flash fee to 4 decimals of precision. For example, 0.0025 ETH = 25. 500 USDC = 5_000_000.
88:        uint56 public changeFee;
89:    
90:        /// @notice The buy/sell fee rate (in basis points) 200 = 2%
91:        uint16 public feeRate;
92:    
93:        /// @notice Whether or not the pool has been initialized.
94:        bool public initialized;
95:    
96:        /// @notice Whether or not the pool pays royalties to the NFT creator on each trade.
97:        bool public payRoyalties;
98:    
99:        /// @notice Whether or not the pool uses the stolen NFT oracle to check if an NFT is stolen.
100:        bool public useStolenNftOracle;
```


### Recommendation
Try to put state variables packed into the same slot

## [G-02] x += y or x -= y costs more gas than x = x + y or x = x - y for state variables

### Impact
Using the addition operator instead of plus-equals saves 113 gas. Usually does not work with struct and mappings.

### Findings
Total:5

[src/PrivatePool.sol#L678](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L678)
```solidity
678:    sum += tokenWeights[i];
```
[src/PrivatePool.sol#L247](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L247)
```solidity
247:    royaltyFeeAmount += royaltyFee;
```
[src/PrivatePool.sol#L341](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L341)
```solidity
341:    royaltyFeeAmount += royaltyFee;
```
[src/PrivatePool.sol#L252](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L252)
```solidity
252:    netInputAmount += royaltyFeeAmount;
```
[src/PrivatePool.sol#L355](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L355)
```solidity
355:    netOutputAmount -= royaltyFeeAmount;
```

## [G-03] Functions guaranteed to revert when called by normal users can be marked payable

### Impact
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
   The extra opcodes avoided are `CALLVALUE(2)`,`DUP1(3)`,`ISZERO(3)`,`PUSH2(3)`,`JUMPI(10)`,`PUSH1(3)`,`DUP1(3)`,`REVERT(0)`,`JUMPDEST(1)`,`POP(2)`, which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### Findings
Total:11

[src/Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol#L148)
```solidity
148:    function withdraw(address token, uint256 amount) public onlyOwner {
```
[src/Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol#L129)
```solidity
129:    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
```
[src/Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol#L135)
```solidity
135:    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
```
[src/Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol#L141)
```solidity
141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
```
[src/PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L587)
```solidity
587:    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```
[src/PrivatePool.sol#L459](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L459)
```solidity
459:    function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
```
[src/PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L550)
```solidity
550:    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
```
[src/PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L576)
```solidity
576:    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
```
[src/PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L562)
```solidity
562:    function setFeeRate(uint16 newFeeRate) public onlyOwner {
```
[src/PrivatePool.sol#L538](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L538)
```solidity
538:    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
```
[src/PrivatePool.sol#L514](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L514)
```solidity
514:    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
```

### Recommendation
Mark the function as payable.


## [G-04] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops

### Impact
The unchecked keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas [per loop](https://gist.github.com/hrkrshnn/ee8fabd532058307229d65dcd5836ddc#the-increment-in-for-loop-post-condition-can-be-made-unchecked). 

### Findings
Total:19

[src/Factory.sol#L119](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol#L119)
```solidity
119:    for (uint256 i = 0; i < tokenIds.length; i++) {
```
[src/EthRouter.sol#L284](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L284)
```solidity
284:    for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
```
[src/EthRouter.sol#L161](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L161)
```solidity
161:    for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
```
[src/EthRouter.sol#L183](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L183)
```solidity
183:    for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
```
[src/EthRouter.sol#L239](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L239)
```solidity
239:    for (uint256 i = 0; i < tokenIds.length; i++) {
```
[src/EthRouter.sol#L106](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L106)
```solidity
106:    for (uint256 i = 0; i < buys.length; i++) {
```
[src/EthRouter.sol#L261](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L261)
```solidity
261:    for (uint256 i = 0; i < changes.length; i++) {
```
[src/EthRouter.sol#L159](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L159)
```solidity
159:    for (uint256 i = 0; i < sells.length; i++) {
```
[src/EthRouter.sol#L265](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L265)
```solidity
265:    for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
```
[src/EthRouter.sol#L116](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L116)
```solidity
116:    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
```
[src/EthRouter.sol#L134](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol#L134)
```solidity
134:    for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
```
[src/PrivatePool.sol#L446](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L446)
```solidity
446:    for (uint256 i = 0; i < outputTokenIds.length; i++) {
```
[src/PrivatePool.sol#L441](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L441)
```solidity
441:    for (uint256 i = 0; i < inputTokenIds.length; i++) {
```
[src/PrivatePool.sol#L238](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L238)
```solidity
238:    for (uint256 i = 0; i < tokenIds.length; i++) {
```
[src/PrivatePool.sol#L272](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L272)
```solidity
272:    for (uint256 i = 0; i < tokenIds.length; i++) {
```
[src/PrivatePool.sol#L329](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L329)
```solidity
329:    for (uint256 i = 0; i < tokenIds.length; i++) {
```
[src/PrivatePool.sol#L496](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L496)
```solidity
496:    for (uint256 i = 0; i < tokenIds.length; i++) {
```
[src/PrivatePool.sol#L518](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L518)
```solidity
518:    for (uint256 i = 0; i < tokenIds.length; i++) {
```
[src/PrivatePool.sol#L673](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L673)
```solidity
673:    for (uint256 i = 0; i < tokenIds.length; i++) {
```


## [G-05] public function that could be declared external

### Impact
`public` functions that are never called by the contract should be declared `external`, and its immutable parameters should be located in calldata to save gas. (see [here](https://github.com/crytic/slither/wiki/Detector-Documentation#public-function-that-could-be-declared-external))

### Findings
Total:12

[src/Factory.sol#L161](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol#L161)
```solidity
161:    function tokenURI(uint256 id) public view override returns (string memory) {
```
[src/PrivatePoolMetadata.sol#L35](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePoolMetadata.sol#L35)
```solidity
35:    function attributes(uint256 tokenId) public view returns (string memory) {
```
[src/PrivatePoolMetadata.sol#L55](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePoolMetadata.sol#L55)
```solidity
55:    function svg(uint256 tokenId) public view returns (bytes memory) {
```
[src/PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L550)
```solidity
550:    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
```
[src/PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L562)
```solidity
562:    function setFeeRate(uint16 newFeeRate) public onlyOwner {
```
[src/PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L576)
```solidity
576:    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
```
[src/PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L587)
```solidity
587:    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```
[src/PrivatePool.sol#L694](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L694)
```solidity
694:    function buyQuote(uint256 outputAmount)
```
[src/PrivatePool.sol#L713](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L713)
```solidity
713:    function sellQuote(uint256 inputAmount)
```
[src/PrivatePool.sol#L731](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L731)
```solidity
731:    function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```
[src/PrivatePool.sol#L742](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L742)
```solidity
742:    function price() public view returns (uint256) {
```
[src/PrivatePool.sol#L755](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol#L755)
```solidity
755:    function flashFeeToken() public view returns (address) {
```

### Recommendation
Use the `external` attribute for functions never called from the contract, and change the location of immutable parameters to `calldata` to save gas.


## [G-06] Optimize names to save gas

### Impact
public/external function names and public member variable names can be optimized to save gas. See [this](https://gist.github.com/IllIllI000/a5d8b486a8259f9f77891a919febd1a9) link for an example of how it works. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call

### Findings
Total: 3

[src/EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/tree/main//src/EthRouter.sol)
```solidity
45:    contract EthRouter is ERC721TokenReceiver {
```

| Function Signature | Function Name |
|--------------------|---------------|
| 0xbf1dab | buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) |
| 0xe87423 | sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) |
| 0xef7354 | deposit(address payable privatePool, address nft,uint256[] calldata tokenIds,uint256 minPrice,uint256 maxPrice,uint256 deadline) |
| 0x4c5785 | change(Change[] calldata changes, uint256 deadline) |
| 0x8cf894 | getRoyalty(address nft, uint256 tokenId, uint256 salePrice) |

[src/Factory.sol](https://github.com/code-423n4/2023-04-caviar/tree/main//src/Factory.sol)
```solidity
37:    contract Factory is ERC721, Owned {
```
| Function Signature | Function Name |
|--------------------|---------------|
| 0xf185b7 | create() |
| 0x89cbe0 | setPrivatePoolMetadata(address _privatePoolMetadata) |
| 0x44e9b0 | setPrivatePoolImplementation(address _privatePoolImplementation) |
| 0x8c4167 | setProtocolFeeRate(uint16 _protocolFeeRate) |
| 0x04d59c | withdraw(address token, uint256 amount) |
| 0x9b3b92 | tokenURI(uint256 id) |
| 0xee0d44 | predictPoolDeploymentAddress(bytes32 salt) |

[src/PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/tree/main//src/PrivatePool.sol)
```solidity
45:    contract PrivatePool is ERC721TokenReceiver {
```
| Function Signature | Function Name |
|--------------------|---------------|
| 0x1afddf | initialize() |
| 0x958711 | buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof) |
| 0x138c57 | sell() |
| 0x731e2f | change() |
| 0xf83540 | execute(address target, bytes memory data) |
| 0xc85c37 | deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) |
| 0x472c6c | withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) |
| 0x6edf26 | setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) |
| 0x6c33e1 | setMerkleRoot(bytes32 newMerkleRoot) |
| 0x0f6823 | setFeeRate(uint16 newFeeRate) |
| 0x9cd0b8 | setUseStolenNftOracle(bool newUseStolenNftOracle) |
| 0x293f0b | setPayRoyalties(bool newPayRoyalties) |
| 0x67c50f | setAllParameters(...) |
| 0x5ec1bd | flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data) |
| 0x60a835 | sumWeightsAndValidateProof(... ) |
| 0x776a58 | buyQuote(uint256 outputAmount) |
| 0x62915d | sellQuote(uint256 inputAmount) |
| 0x6c3376 | changeFeeQuote(uint256 inputAmount) |
| 0xa035b1 | price() |
| 0x81890b | flashFee(address, uint256) |
| 0xe3afca | flashFeeToken() |
| 0x496903 | availableForFlashLoan(address token, uint256 tokenId) |