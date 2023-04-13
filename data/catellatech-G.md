# Summary
|ID     | Optimization Details|  Gas saved |Instances|
|:----:  | :---           |  :----:         |:----:         |
| [G-01] | DUPLICATED REQUIRE()/IF() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION | 688 | 12 |
| [G-02] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO)| 1017 | 9 |
| [G-03] | EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING | - | 3 |
| [G-04] | OPTIMIZING EVENT PARAMETERS: INDEXING WHERE POSSIBLE | 1059 | 8 |
| [G-05] | USE DOBLE IF INSTEAD && | 4 | 7 |
| [G-06] | USE SHIFT RIGHT/LEFT INSTEAD OF DIVISION/MULTIPLICATION | 350 | 2 |
| [G-07] | USE NESTED IF AND, AVOID MULTIPLE CHECK COMBINATIONS | 550 | 11 |
| [G-08] | USE ASSEMBLY TO CHECK FOR ADDRESS(0) | 340 | 20 |
| [G-09] | SETTING THE CONSTRUCTOR TO PAYABLE | 39 | 3 |
| [G-10] | FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE | 231 | 11 |
| [G-11] | USE HARDCODED ADDRESS INSTEAD OF ADDRESS(THIS) | - | 24 |
| [G-12] | OPTIMIZE NAMES TO SAVE GAS | 110 | 5 |
| [G-13] | PUBLIC FUNCTIONS TO EXTERNAL | - | 34 |

| Gas saved | 4384 | Total Instances | 140 |
|:--:|:--:|:--:|--:|

# Detailed Findings
## [G-01] DUPLICATED REQUIRE()/IF() CHECKS SHOULD BE REFACTORED TO A MODIFIER OR FUNCTION 
```solidity
main/src/EthRouter.sol

// @audit Add a modifier

  101: if (block.timestamp > deadline && deadline != 0) {
  154: if (block.timestamp > deadline && deadline != 0) {
  228: if (block.timestamp > deadline && deadline != 0) {
  256: if (block.timestamp > deadline && deadline != 0) {
```
- [EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101) 
- [EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154)
- [EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228)
- [EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256)

```solidity
main/src/PrivatePool.sol

// @audit Add a modifier

  225: if (baseToken != address(0) && msg.value > 0)
  397: if (baseToken != address(0) && msg.value > 0)

  254: if (baseToken != address(0)) {
  278: if (baseToken != address(0)) {
  345: if (baseToken != address(0)) {
  421: if (baseToken != address(0)) {
  500: if (baseToken != address(0)) {
  651: if (baseToken != address(0))         
```
- [PrivatePool.sol#225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225) 
- [PrivatePool.sol#397](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L497) 
- [PrivatePool.sol#254](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L254) 
- [PrivatePool.sol#278](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L278) 
- [PrivatePool.sol#345](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345) 
- [PrivatePool.sol#421](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L421) 
- [PrivatePool.sol#500](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L500) 
- [PrivatePool.sol#651](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651) 

### RECOMMENDATION
One way to eliminate duplicated require() and if() checks in your Solidity code is to refactor them into a modifier. This can help to reduce code duplication and improve the readability and maintainability of your code.

To do this, you can create a modifier that includes the require() or if() check, and then apply the modifier wherever the check is needed in your code. This way, if you need to make changes to the check later on, you only need to do it in one place, rather than in multiple places throughout your code.

For example, you could create a modifier like this

```solidity
main/src/EthRouter.sol#L101

    modifier checkDeadline(uint256 deadline) {
        if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
        _;
    }
```
Here are the data available in the covered contracts. Taking advantage of this situation in non-covered contracts can also lead to gas consumption optimization.

## [G-02] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO)
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/catellaTech/8539f7345b17f0929a41598e7e00e3c2) in Solidity. The same applies to the subtraction operation.

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

## [G-03] EMPTY BLOCKS SHOULD BE REMOVED OR EMIT SOMETHING
It is recommended to refactor the code to eliminate empty blocks, or to make the block perform some useful action, such as emitting an event or reverting. If the contract is meant to be extended, it should be declared as abstract and the function signatures should be added without any default implementation.

If the block is an empty if-statement block used to avoid performing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement. This is because the else-if/else conditions involve different classes of checks, which may lead to the introduction of errors when the code is later modified. For example, the structure if(x){}else if(y){...}else{...} should be replaced with if(!x){if(y){...}else{...}}.

Finally, empty receive()/fallback() functions with payable that are not used should be removed to save deployment gas on the blockchain.

```solidity
// @audit Refactorize
    main/src/Factory.sol:
    55: receive() external payable {}

    main/src/EthRouter.sol
    88: receive() external payable {}

    main/src/PrivatePool.sol
    134: receive() external payable {}
```
- [Factory.sol#L55](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L55)
- [EthRouter.sol#L88](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88)
- [PrivatePool.sol#L134](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L134)

## [G-04] OPTIMIZING EVENT PARAMETERS: INDEXING WHERE POSSIBLE
When creating events in Solidity, it is best practice to index the event parameters for efficient gas usage. The most gas-efficient approach is to index up to three event parameters. If there are fewer than three parameters, then it is recommended to index all of them. This will optimize the search and filtering functions of the Ethereum event logs. It is important to strike a balance between efficiency and functionality.

- [Factory.sol#L41](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L41)

```diff
-    event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);
+    event Create(address indexed privatePool, uint256[] indexed tokenIds, uint256 indexed baseTokenAmount);
```
The same optimization of adding an extra indexed parameter can be applied to:
- [PrivatePool.sol#L61](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L61)
- [PrivatePool.sol#L62](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L62)
- [PrivatePool.sol#L64](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L64)
- [PrivatePool.sol#L65](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L65)
- [PrivatePool.sol#L66](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L66)
- [PrivatePool.sol#L67](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L67)
- [PrivatePool.sol#L68](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L68)

## [G-05] USE DOBLE IF EN LUGAR DE &&
When the condition of the 'if' statement includes a logical 'AND' operator and is not followed by an 'else' statement, it is preferable to replace it with two separate 'if' statements. This not only makes the code more readable and easier to understand, but it can also be more optimal in terms of performance and resource consumption, as it avoids unnecessary evaluation of the second condition if the first one is false.

`main/src/Factory.sol`
- [Factory.sol#L87](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87)

`main/src/EthRouter.sol`
- [EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101)
- [EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154)
- [EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228)
- [EthRouter.sol#L234](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L234)
- [EthRouter.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256)

`main/src/PrivatePool.sol`
- [PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225)

## [G-06] USE SHIFT RIGHT/LEFT INSTEAD OF DIVISION/MULTIPLICATION
It is possible to calculate a division or multiplication by any power of 2 number x by shifting right or left, instead of using division or multiplication operations. This can be useful in terms of performance and gas consumption, as the shift operation SHR only uses 3 gas units, while the DIV operation uses 5 units.

In addition, Solidity's division operation includes a division-by-zero prevention that can further increase the cost of the operation. Using shifts instead of division can avoid this prevention and reduce gas consumption.

```diff
main/src/PrivatePool.sol:
- 703: protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() / 10_000;
+ 703: protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() >> 10_000;

- 704: feeAmount = inputAmount * feeRate / 10_000;
+ 703: feeAmount = inputAmount * feeRate >> 10_000;
```
- [PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L703-L704)
- [PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L721-L722)

## [G-07] USE NESTED IF AND, AVOID MULTIPLE CHECK COMBINATIONS
Using nested "if" statements instead of multiple "&&" check combinations can be cheaper in terms of gas consumption, but there are also additional advantages to consider. For example, nested "if" statements can make the code easier to read and understand, as well as provide better coverage reports. By breaking up complex logical checks into smaller, more easily digestible pieces, the code can be more maintainable and less error-prone over time. Additionally, using nested "if" statements can help avoid unexpected interactions between multiple conditions, leading to fewer bugs and more reliable code.

```solidity
main/src/EthRouter.sol
    87: if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
    101: if (block.timestamp > deadline && deadline != 0) {
    154: if (block.timestamp > deadline && deadline != 0) {
    228: if (block.timestamp > deadline && deadline != 0) {
    256: if (block.timestamp > deadline && deadline != 0) {

main/src/PrivatePool.sol
    225: if (baseToken != address(0) && msg.value > 0) 
    277: if (royaltyFee > 0 && recipient != address(0))
    344: if (royaltyFee > 0 && recipient != address(0))
    397: if (baseToken != address(0) && msg.value > 0) 
    489: if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0)))
    635: if (baseToken == address(0) && msg.value < fee)
```
### RECOMENDATION
```diff
main/src/EthRouter.sol#L101
- 101:    if (block.timestamp > deadline && deadline != 0) {
+         if (block.timestamp > deadline) {
+           if (deadline != 0) {
+           }
+         } 
```

## [G-08] USE ASSEMBLY TO CHECK FOR ADDRESS(0)
Saves 6 gas per instance if using assembly to check for address(0).

```solidity
main/src/Factory.sol
87:  if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) 
110: if (_baseToken == address(0)) 
149: if (token == address(0))

main/src/PrivatePoolMetadata.sol
47: trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) 
103: "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) 


main/src/PrivatePool.sol
    225: if (baseToken != address(0) && msg.value > 0) 
    154: if (baseToken != address(0))
    277: if (royaltyFee > 0 && recipient != address(0))
    278: if (baseToken != address(0))
    344: if (royaltyFee > 0 && recipient != address(0))
    345: if (baseToken != address(0))
    397: if (baseToken != address(0) && msg.value > 0) 
    421: if (baseToken != address(0))
    489: if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0)))
    522: if (token == address(0))
    635: if (baseToken == address(0) && msg.value < fee)
    733: uint256 exponent = baseToken == address(0) ? ...
    744: uint256 exponent = baseToken == address(0) ? ...
```
### RECOMENDATION
```solidity
// @audit example
assembly {
 if iszero(_addr) {
  mstore(0x00, "zero address")
  revert(0x00, 0x20)
 }
}
```
## [G-09] SETTING THE CONSTRUCTOR TO PAYABLE
Saves ~13 gas per instance

```solidity
main/src/Factory.sol
53: constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}

main/src/EthRouter.sol
90:constructor(address _royaltyRegistry) {

main/src/PrivatePool.sol
143: constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
```

## [G-10] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE
If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### PROOF OF CONCEPT
```solidity
main/src/Factory.sol
    129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
    135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
    141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
    148: function withdraw(address token, uint256 amount) public onlyOwner {

main/src/PrivatePool.sol
    459: function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
    514: function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
    538: function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
    550: function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
    562: function setFeeRate(uint16 newFeeRate) public onlyOwner {
    576: function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
    587: function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```
### RECOMMENDATION
Functions guaranteed to revert when called by normal users can be marked payable.

## [G-11] USE HARDCODED ADDRESS INSTEAD OF ADDRESS(THIS)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's script.sol and solmate's LibRlp.sol contracts can help achieve this.

[Reference](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

https://twitter.com/transmissions11/status/1518507047943245824

### PROOF OF CONCEPT
```solidity
main/src/Factory.sol
    169: predictedAddress = privatePoolImplementation.predictDeterministicAddress(salt, address(this));

main/src/EthRouter.sol
    136: ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
    141: if (address(this).balance > 0) {
                msg.sender.safeTransferETH(address(this).balance);
            }

    162: ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
    203: if (address(this).balance < minOutputAmount) {
    208: msg.sender.safeTransferETH(address(this).balance);
    240: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
    266: ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);
    285: ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);
    290: if (address(this).balance > 0) {
                msg.sender.safeTransferETH(address(this).balance);
            }

main/src/PrivatePool.sol
    128: if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
    240: ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
    256: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);
    331: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
    423: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);
    442: ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
    447: ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
    497: ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
    502: ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);
    519: ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
    638: ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);
    648: ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);
    651: if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee)
    766: return result == address(this);
```
### RECOMMENDATION
Use hardcoded address

## [G-12] OPTIMIZE NAMES TO SAVE GAS
Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

### PROOF OF CONCEPT
All scope.

### RECOMMENDATION
Find a lower method ID name for the most called functions for example Call() vs. Call1() is cheaper by 22 gas For example, the function IDs in the Gauge.sol contract will be the most used; A lower method ID may be given.

## [G-13] PUBLIC FUNCTIONS TO EXTERNAL
The following functions could be set external to save gas and improve code quality. External call cost is less expensive than of public functions.

### PROOF OF CONCEPT
```solidity
main/src/Factory.sol
    129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
    135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
    141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
    148: function withdraw(address token, uint256 amount) public onlyOwner {
    161: function tokenURI(uint256 id) public view override returns (string memory) {
    168: function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

main/src/PrivatePoolMetadata.sol
    17: function tokenURI(uint256 tokenId) public view returns (string memory) {
    35: function attributes(uint256 tokenId) public view returns (string memory) {
    55:  function svg(uint256 tokenId) public view returns (bytes memory) {

main/src/EthRouter.sol
    99: function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
    152: function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public {
    219: function deposit(
            address payable privatePool,
            address nft,
            uint256[] calldata tokenIds,
            uint256 minPrice,
            uint256 maxPrice,
            uint256 deadline
        ) public payable {

    254: function change(Change[] calldata changes, uint256 deadline) public payable {
    301: function getRoyalty(address nft, uint256 tokenId, uint256 salePrice)
            public
            view
            returns (uint256 royaltyFee, address recipient)
        {

main/src/PrivatePool.sol
    157: function initialize(
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

    211: function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
            public
            payable
            returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
        {

    301: function sell(
            uint256[] calldata tokenIds,
            uint256[] calldata tokenWeights,
            MerkleMultiProof calldata proof,
            IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
        ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {

    385: function change(
            uint256[] memory inputTokenIds,
            uint256[] memory inputTokenWeights,
            MerkleMultiProof memory inputProof,
            IStolenNftOracle.Message[] memory stolenNftProofs,
            uint256[] memory outputTokenIds,
            uint256[] memory outputTokenWeights,
            MerkleMultiProof memory outputProof
        ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {

    395: function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
    484: function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {
    414: function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
    538: function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
    550: function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
    562: function setFeeRate(uint16 newFeeRate) public onlyOwner {
    576: function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
    587: function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
    602: function setAllParameters(
            uint128 newVirtualBaseTokenReserves,
            uint128 newVirtualNftReserves,
            bytes32 newMerkleRoot,
            uint16 newFeeRate,
            bool newUseStolenNftOracle,
            bool newPayRoyalties
        ) public {

    661: function sumWeightsAndValidateProof(
            uint256[] memory tokenIds,
            uint256[] memory tokenWeights,
            MerkleMultiProof memory proof
        ) public view returns (uint256) {

    694: function buyQuote(uint256 outputAmount)
            public
            view
            returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
        {

    713: function sellQuote(uint256 inputAmount)
            public
            view
            returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
        {

    731: function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
    742: function price() public view returns (uint256) {
    750: function flashFee(address, uint256) public view returns (uint256) {
    755: function flashFeeToken() public view returns (address) {
    763:  function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {
```