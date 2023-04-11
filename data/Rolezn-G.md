## Summary<a name="Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | `abi.encode()` is less efficient than `abi.encodepacked()` | 2 | 200 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Use calldata instead of memory for function parameters | 4 | 1200 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Setting the `constructor` to `payable` | 3 | 39 |
| [GAS&#x2011;4](#GAS&#x2011;4) | Using `delete` statement can save gas | 2 | - |
| [GAS&#x2011;5](#GAS&#x2011;5) | Functions guaranteed to revert when called by normal users can be marked `payable` | 11 | 231 |
| [GAS&#x2011;6](#GAS&#x2011;6) | Use hardcoded address instead `address(this)` | 25 | - |
| [GAS&#x2011;7](#GAS&#x2011;7) | Optimize names to save gas | 4 | 88 |
| [GAS&#x2011;8](#GAS&#x2011;8) | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 9 | - |
| [GAS&#x2011;9](#GAS&#x2011;9) | Public Functions To External | 32 | - |
| [GAS&#x2011;10](#GAS&#x2011;10) | Shorten the array rather than copying to a new one | 1 | - |
| [GAS&#x2011;11](#GAS&#x2011;11) | String literals passed to `abi.encode()`/`abi.encodePacked()` should not be split by commas | 58 | 1218 |
| [GAS&#x2011;12](#GAS&#x2011;12) | Structs can be packed into fewer storage slots | 2 | 4000 |
| [GAS&#x2011;13](#GAS&#x2011;13) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead | 5 | - |

Total: 158 contexts over 13 issues

## Gas Optimizations

### <a href="#Summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> `abi.encode()` is less efficient than `abi.encodepacked()`

See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison 

#### <ins>Proof Of Concept</ins>


```solidity
177: abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L177

```solidity
675: leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L675






### <a href="#Summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Use calldata instead of memory for function parameters

In some cases, having function arguments in calldata instead of
memory is more optimal.

Consider the following generic example:
```
contract C {
    function add(uint[] memory arr) external returns (uint sum) {
        uint length = arr.length;
        for (uint i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }
}
```
In the above example, the dynamic array arr has the storage location
memory. When the function gets called externally, the array values are
kept in calldata and copied to memory during ABI decoding (using the
opcode calldataload and mstore). And during the for loop, arr[i]
accesses the value in memory using a mload. However, for the above
example this is inefficient. Consider the following snippet instead:
```
contract C {
    function add(uint[] calldata arr) external returns (uint sum) {
        uint length = arr.length;
        for (uint i = 0; i < arr.length; i++) {
            sum += arr[i];
        }
    }
}
```
In the above snippet, instead of going via memory, the value is directly
read from calldata using calldataload. That is, there are no
intermediate memory operations that carries this value.

Gas savings: In the former example, the ABI decoding begins with
copying value from calldata to memory in a for loop. Each iteration
would cost at least 60 gas. In the latter example, this can be
completely avoided. This will also reduce the number of instructions and
therefore reduces the deploy time cost of the contract.

In short, use calldata instead of memory if the function argument
is only read.

Note that in older Solidity versions, changing some function arguments
from memory to calldata may cause "unimplemented feature error".
This can be avoided by using a newer (0.8.*) Solidity compiler.

Examples
Note: The following pattern is prevalent in the codebase:
```
function f(bytes memory data) external {
    (...) = abi.decode(data, (..., types, ...));
}
```
Here, changing to bytes calldata will decrease the gas. The total
savings for this change across all such uses would be quite
significant.

#### <ins>Proof Of Concept</ins>


```solidity
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
        uint256[] memory tokenIds, 
        uint256 baseTokenAmount
    ) public payable returns (PrivatePool privatePool) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L71

```solidity
function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs 
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L301

```solidity
function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L385

```solidity
function sumWeightsAndValidateProof(
        uint256[] memory tokenIds,
        uint256[] memory tokenWeights,
        MerkleMultiProof memory proof
    ) public view returns (uint256) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L661






### <a href="#Summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Setting the `constructor` to `payable`

Saves ~13 gas per instance

#### <ins>Proof Of Concept</ins>

```solidity
90: constructor(address _royaltyRegistry)
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L90

```solidity
53: constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender)
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L53

```solidity
143: constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle)
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L143





### <a href="#Summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Using `delete` statement can save gas

#### <ins>Proof Of Concept</ins>

```solidity
237: uint256 royaltyFeeAmount = 0;

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L237

```solidity
328: uint256 royaltyFeeAmount = 0;

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L328





### <a href="#Summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### <ins>Proof Of Concept</ins>

```solidity
129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L129

```solidity
135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L135

```solidity
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L141

```solidity
148: function withdraw(address token, uint256 amount) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L148

```solidity
459: function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L459

```solidity
514: function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L514

```solidity
538: function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L538

```solidity
550: function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L550

```solidity
562: function setFeeRate(uint16 newFeeRate) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L562

```solidity
576: function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L576

```solidity
587: function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L587



#### <ins>Recommended Mitigation Steps</ins>
Functions guaranteed to revert when called by normal users can be marked payable.



### <a href="#Summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry's script.sol and solmate's `LibRlp.sol` contracts can help achieve this.

References: 
https://book.getfoundry.sh/reference/forge-std/compute-create-address 

https://twitter.com/transmissions11/status/1518507047943245824

#### <ins>Proof Of Concept</ins>

```solidity
136: ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
141: if (address(this).balance > 0) {
142: msg.sender.safeTransferETH(address(this).balance);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L136

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L141

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L142



```solidity
162: ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
203: if (address(this).balance < minOutputAmount) {
208: msg.sender.safeTransferETH(address(this).balance);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L162

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L203

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L208



```solidity
240: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L240

```solidity
266: ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);
285: ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);
290: if (address(this).balance > 0) {
291: msg.sender.safeTransferETH(address(this).balance);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L266

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L285

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L290

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L291



```solidity
169: predictedAddress = privatePoolImplementation.predictDeterministicAddress(salt, address(this));

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L169

```solidity
240: ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
256: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L240

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L256



```solidity
331: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L331

```solidity
423: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);
442: ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
447: ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L423

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L442

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L447



```solidity
497: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
502: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L497

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L502



```solidity
519: ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L519

```solidity
638: ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);
648: ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);
651: if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L638

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L648

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L651



```solidity
766: return result == address(this);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L766



#### <ins>Recommended Mitigation Steps</ins>

Use hardcoded `address`





### <a href="#Summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions. 

See more <a href="https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92">here</a>

#### <ins>Proof Of Concept</ins>

All in-scope contracts

#### <ins>Recommended Mitigation Steps</ins>
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas
For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.



### <a href="#Summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

#### <ins>Proof Of Concept</ins>


```solidity
230: virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
231: virtualNftReserves -= uint128(weightSum);
247: royaltyFeeAmount += royaltyFee;
252: netInputAmount += royaltyFeeAmount;

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L230

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L231

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L247

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L252



```solidity
323: virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
324: virtualNftReserves += uint128(weightSum);
341: royaltyFeeAmount += royaltyFee;
355: netOutputAmount -= royaltyFeeAmount;

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L323

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L324

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L341

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L355



```solidity
678: sum += tokenWeights[i];

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L678








### <a href="#Summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

#### <ins>Proof Of Concept</ins>


```solidity
function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L99

```solidity
function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L152

```solidity
function deposit(
        address payable privatePool,
        address nft,
        uint256[] calldata tokenIds,
        uint256 minPrice,
        uint256 maxPrice,
        uint256 deadline
    ) public payable {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L219

```solidity
function change(Change[] calldata changes, uint256 deadline) public payable {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L254

```solidity
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
        uint256[] memory tokenIds, 
        uint256 baseTokenAmount
    ) public payable returns (PrivatePool privatePool) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L71

```solidity
function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L129

```solidity
function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L135

```solidity
function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L141

```solidity
function withdraw(address token, uint256 amount) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L148

```solidity
function tokenURI(uint256 id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L161

```solidity
function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L168

```solidity
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

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L157

```solidity
function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs 
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L301

```solidity
function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L385

```solidity
function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L459

```solidity
function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L484

```solidity
function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L514

```solidity
function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L538

```solidity
function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L550

```solidity
function setFeeRate(uint16 newFeeRate) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L562

```solidity
function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L576

```solidity
function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L587

```solidity
function setAllParameters(
        uint128 newVirtualBaseTokenReserves,
        uint128 newVirtualNftReserves,
        bytes32 newMerkleRoot,
        uint16 newFeeRate,
        bool newUseStolenNftOracle,
        bool newPayRoyalties
    ) public {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L602

```solidity
function sumWeightsAndValidateProof(
        uint256[] memory tokenIds,
        uint256[] memory tokenWeights,
        MerkleMultiProof memory proof
    ) public view returns (uint256) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L661

```solidity
function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L731

```solidity
function price() public view returns (uint256) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L742

```solidity
function flashFee(address, uint256) public view returns (uint256) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L750

```solidity
function flashFeeToken() public view returns (address) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L755

```solidity
function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L763

```solidity
function tokenURI(uint256 tokenId) public view returns (string memory) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L17

```solidity
function attributes(uint256 tokenId) public view returns (string memory) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L35

```solidity
function svg(uint256 tokenId) public view returns (bytes memory) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L55






### <a href="#Summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

#### <ins>Proof Of Concept</ins>


```solidity
672: bytes32[] memory leafs = new bytes32[](tokenIds.length);

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L672





### <a href="#Summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> String literals passed to `abi.encode()`/`abi.encodePacked()` should not be split by commas

String literals can be split into multiple parts and still be considered as a single string literal. Adding commas between each chunk makes it no longer a single string, and instead multiple strings. EACH new comma costs 21 gas due to stack operations and separate MSTOREs.

#### <ins>Proof Of Concept</ins>


```solidity
19: bytes memory metadata = abi.encodePacked(
            "{",
                '"name": "Private Pool ',Strings.toString(tokenId),'",',
                '"description": "Caviar private pool AMM position.",',
                '"image": ','"data:image/svg+xml

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L19

```solidity
39: bytes memory _attributes = abi.encodePacked(
            trait("Pool address", Strings.toHexString(address(privatePool))), ',',
            trait("Base token", Strings.toHexString(privatePool.baseToken())), ',',
            trait("NFT", Strings.toHexString(privatePool.nft())), ',',
            trait("Virtual base token reserves",Strings.toString(privatePool.virtualBaseTokenReserves())), ',',
            trait("Virtual NFT reserves", Strings.toString(privatePool.virtualNftReserves())), ',',
            trait("Fee rate (bps): ", Strings.toString(privatePool.feeRate())), ',',
            trait("NFT balance", Strings.toString(ERC721(privatePool.nft()).balanceOf(address(privatePool)))), ',',
            trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))
        )

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L39

```solidity
62: _svg = abi.encodePacked(
                '<svg xmlns="http:
                    '<text x="24px" y="24px" font-size="12">',
                        "Caviar AMM private pool position",
                    "</text>",
                    '<text x="24px" y="48px" font-size="12">',
                        "Private pool: ", Strings.toHexString(address(privatePool)),
                    "</text>",
                    '<text x="24px" y="72px" font-size="12">',
                        "Base token: ", Strings.toHexString(privatePool.baseToken()),
                    "</text>",
                    '<text x="24px" y="96px" font-size="12">',
                        "NFT: ", Strings.toHexString(privatePool.nft()),
                    "</text>"
            )
81: _svg = abi.encodePacked(
                _svg,
                '<text x="24px" y="120px" font-size="12">',
                    "Virtual base token reserves: ", Strings.toString(privatePool.virtualBaseTokenReserves()),
                "</text>",
                '<text x="24px" y="144px" font-size="12">',
                    "Virtual NFT reserves: ", Strings.toString(privatePool.virtualNftReserves()),
                "</text>",
                '<text x="24px" y="168px" font-size="12">',
                    "Fee rate (bps): ", Strings.toString(privatePool.feeRate()),
                "</text>"
            )
97: _svg = abi.encodePacked(
                _svg, 
                    '<text x="24px" y="192px" font-size="12">',
                        "NFT balance: ", Strings.toString(ERC721(privatePool.nft()).balanceOf(address(privatePool))),
                    "</text>",
                    '<text x="24px" y="216px" font-size="12">',
                        "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),
                    "</text>",
                "</svg>"
            )

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L62

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L81

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L97



```solidity
115: abi.encodePacked(
                '{ "trait_type": "', traitType, '",', '"value": "', value, '" }'
            )
        )

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L115





### <a href="#Summary">[GAS&#x2011;12]</a><a name="GAS&#x2011;12"> Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

#### <ins>Proof Of Concept</ins>

```solidity
69: struct Change {
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

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L69

Can potentially save starting from 2 gas slots if we do not expect `TokenIds` and `TokenWeights` to ever exceed `uint128`.




### <a href="#Summary">[GAS&#x2011;13]</a><a name="GAS&#x2011;13"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


```solidity
51: uint16 public protocolFeeRate;
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L51

```solidity
88: uint56 public changeFee;
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L88

```solidity
91: uint16 public feeRate;
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L91

```solidity
104: uint128 public virtualBaseTokenReserves;
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L104

```solidity
112: uint128 public virtualNftReserves;
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L112



