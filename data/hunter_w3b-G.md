# Gas Optimization

# Summary

| Number | Optimization Details                                                                         | Context |
| :----: | :------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE             |   10    |
| [G-02] |  <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES                          |    4    |
| [G-03] | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD              |    9    |
| [G-04] | SETTING THE CONSTRUCTOR TO PAYABLE                                                           |    3    |
| [G-05] | STATE VARIABLES SHOULD BE CACHED IN STACK VARIABLES RATHER THAN RE-READING THEM FROM STORAGE |   14    |
| [G-06] | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS          |    9    |
| [G-07] | USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS     |    8    |
| [G-08] | USE ASSEMBLY TO CHECK FOR ADDRESS(0)                                                         |   19    |
| [G-09] | The result of function calls should be cached rather than re-calling the function            |    3    |
| [G-10] | REORDER STRUCTURE LAYOUT                                                                     |    1    |
| [G-11] | With assembly, .call (bool success)  transfer can be done gas-optimized                      |    1    |
| [G-12] | Duplicated require()/if() checks should be refactored to a modifier or function              |   29    |
| [G-13] | Use nested if and, avoid multiple check combinations                                         |   11    |
| [G-14] | Use assembly to write address storage values                                                 |    8    |
| [G-15] | abi.encodePacked() efficient then abi.encode()                                               |    1    |
| [G-16] | Access mappings directly rather than using accessor functions                                |    2    |

## [G-01] FUNCTIONS GUARANTEED TO REVERT WHEN CALLED BY NORMAL USERS CAN BE MARKED PAYABLE

### Details

The onlyOwner modifier makes a function revert if not called by the address registered as the owner

There are **10** instances of this issue.

### Github Permalinks

```solidity
File: /src/Factory.sol

129    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148    function withdraw(address token, uint256 amount) public onlyOwner {

```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
File: /src/PrivatePool.sol

514    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562    function setFeeRate(uint16 newFeeRate) public onlyOwner {

576    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

587    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-02] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES

### Summary

AVOID COMPOUND ASSIGNMENT OPERATOR IN STATE VARIABLES

Using compound assignment operators for state variables (like State += X or State -= X …) it’s more expensive than using operator assignment (like State = State + X or State = State - X …).

### Details

There are **4** instances of this issue.

### Github Permalinks

```solidity
File: /src/PrivatePool.sol

230        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231        virtualNftReserves -= uint128(weightSum);

323        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324        virtualNftReserves += uint128(weightSum);
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-03] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD

### Summary

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.
public functions not called in the contract itself should be declared externals.

https://github.com/prepo-io/prepo-monorepo/blob/feat/2022-12-prepo/packages/prepo-shared-contracts/contracts/AllowedMsgSenders.sol#L15

### Details

There are **9** instances of this issue.

### Github Permalinks

```solidity
File: /src/Factory.sol

129    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
File: /src/PrivatePoolMetadata.sol

17    function tokenURI(uint256 tokenId) public view returns (string memory) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol

```solidity
File: /src/EthRouter.sol

99    function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {

226    ) public payable {

254    function change(Change[] calldata changes, uint256 deadline) public payable {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
File: /src/PrivatePool.sol

742    function price() public view returns (uint256) {

755    function flashFeeToken() public view returns (address) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-04] SETTING THE CONSTRUCTOR TO PAYABLE

### Details

There are **3** instances of this issue.

### Github Permalinks

```solidity
File: /src/Factory.sol

53    constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
File: /src/EthRouter.sol

90    constructor(address _royaltyRegistry) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
File: /src/PrivatePool.sol

143    constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-05]  STATE VARIABLES SHOULD BE CACHED IN STACK VARIABLES RATHER THAN RE-READING THEM FROM STORAGE

### Summary

The instances below point to the second+ access of a state variable within a function.
Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read.
Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
Most of the times this if statement will be true and we will save 100 gas at a small possibility of 3 gas loss ,

### Details

There are **14** instances of this issue.

### Github Permalinks

```solidity
File: /src/PrivatePool.sol

225        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

254        if (baseToken != address(0)) {

278                    if (baseToken != address(0)) {

345                    if (baseToken != address(0)) {

357        if (baseToken == address(0)) {

397        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

421        if (baseToken != address(0)) {

489        if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

500        if (baseToken != address(0)) {

635        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

651        if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-06]  NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

### Details

Do not use return at the end of the function:

There are **14** instances of this issue.

### Github Permalinks

```solidity
File: /src/Factory.sol

168    function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
File: /src/EthRouter.sol

304        returns (uint256 royaltyFee, address recipient)
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
File: /src/PrivatePool.sol

214        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

306    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {

393    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {

697        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

716        returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

731    function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {

781        returns (uint256 royaltyFee, address recipient)
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-07] USING CALLDATA INSTEAD OF MEMORY FOR READ-ONLY ARGUMENTS IN EXTERNAL FUNCTIONS SAVES GAS

### Summary

calldata must be used when declaring an external function's dynamic parameters

When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 \* <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.
https://github.com/prepo-io/prepo-monorepo/blob/feat/2022-12-prepo/apps/smart-contracts/core/contracts/DepositHook.sol#L75

### Details

There are **8** instances of this issue.

### Github Permalinks

```solidity
File: /src/PrivatePool.sol

386        uint256[] memory inputTokenIds,

387        uint256[] memory inputTokenWeights,

389        IStolenNftOracle.Message[] memory stolenNftProofs,

390        uint256[] memory outputTokenIds,

391        uint256[] memory outputTokenWeights,

662        uint256[] memory tokenIds,

663        uint256[] memory tokenWeights,

664        MerkleMultiProof memory proof
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-08]  USE ASSEMBLY TO CHECK FOR ADDRESS(0)

### Details

There are **19** instances of this issue.

### Github Permalinks

```solidity
File: /src/Factory.sol

87        if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

110        if (_baseToken == address(0)) {

149        if (token == address(0)) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
File: /src/PrivatePool.sol

225        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

254        if (baseToken != address(0)) {

277                if (royaltyFee > 0 && recipient != address(0)) {

278                    if (baseToken != address(0)) {

344                if (royaltyFee > 0 && recipient != address(0)) {

345                    if (baseToken != address(0)) {

357        if (baseToken == address(0)) {

397        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

421        if (baseToken != address(0)) {

489        if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

500        if (baseToken != address(0)) {

522        if (token == address(0)) {

635        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

651        if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

733        uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;

744        uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-09] The result of function calls should be cached rather than re-calling the function

### Details

The instances below point to the second+ call of the function within a single function

There are **3** instances of this issue.

### Github Permalinks

```solidity
File: /src/EthRouter.sol

309        if (IERC2981(lookupAddress).supportsInterface(type(IERC2981).interfaceId)) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
File: /src/PrivatePool.sol

629        if (!availableForFlashLoan(token, tokenId)) revert NotAvailableForFlashLoan();

786        if (IERC2981(lookupAddress).supportsInterface(type(IERC2981).interfaceId)) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-10] REORDER STRUCTURE LAYOUT

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /src/interfaces/IStolenNftOracle.sol

6    struct Message {
        bytes32 id;
        bytes payload;
        // The UNIX timestamp when the message was signed by the oracle
        uint256 timestamp;
        // ECDSA signature or EIP-2098 compact signature
        bytes signature;
    }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/interfaces/IStolenNftOracle.sol#L6

### Recommendation Code

```solidity
File:
    struct Message {
        bytes32 id;
        bytes payload;
        // ECDSA signature or EIP-2098 compact signature
        bytes signature;
        // The UNIX timestamp when the message was signed by the oracle
        uint256 timestamp;
    }
```

## [G-11]  With assembly, .call (bool success)  transfer can be done gas-optimized

### Summary

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0),
this storage disappears and gas optimization is provided.
https://code4rena.com/reports/2023-01-biconomy#g-01-with-assembly-call-bool-success-transfer-can-be-done-gas-optimized

Did you know that the normal "(bool success, bytes memory returnData) = http://target.call()" automatically copies the return data to memory even if you omit the returnData variable? If a relayer executes transactions with such calls it can lead to a gas-griefing attack.

https://t.co/dHtRfZXEnq

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /src/PrivatePool.sol

461         (bool success, bytes memory returnData) = target.call{value: msg.value}(data);
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-12]  Duplicated require()/if() checks should be refactored to a modifier or function

### Details

There are **29** instances of this issue.

### Github Permalinks

```solidity
File: /src/EthRouter.sol

101        if (block.timestamp > deadline && deadline != 0) {
154        if (block.timestamp > deadline && deadline != 0) {
228        if (block.timestamp > deadline && deadline != 0) {
256        if (block.timestamp > deadline && deadline != 0) {

114                if (payRoyalties) {
181                if (payRoyalties) {

121                        if (royaltyFee > 0) {
188                        if (royaltyFee > 0) {

141        if (address(this).balance > 0) {
290        if (address(this).balance > 0) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
File: /src/PrivatePool.sol

225        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
397        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

242            if (payRoyalties) {
271            if (payRoyalties) {
333            if (payRoyalties) {

254        if (baseToken != address(0)) {
278        if (baseToken != address(0)) {
345        if (baseToken != address(0)) {
421        if (baseToken != address(0)) {
500        if (baseToken != address(0)) {

259            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
368            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

265            if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
362            if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
432            if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-13] Use nested if and, avoid multiple check combinations

### Details

There are **11** instances of this issue.

### Github Permalinks

```solidity
File: /src/Factory.sol

87        if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
File: /src/EthRouter.sol

101        if (block.timestamp > deadline && deadline != 0) {

154        if (block.timestamp > deadline && deadline != 0) {

228        if (block.timestamp > deadline && deadline != 0) {

256        if (block.timestamp > deadline && deadline != 0) {
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
File: /src/PrivatePool.sol

225        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

277                if (royaltyFee > 0 && recipient != address(0)) {

344                if (royaltyFee > 0 && recipient != address(0)) {

397        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

489        if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

635        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-14] Use assembly to write address storage values

### Details

There are **8** instances of this issue.

### Github Permalinks

```solidity
File: /src/Factory.sol

130        privatePoolMetadata = _privatePoolMetadata;

136        privatePoolImplementation = _privatePoolImplementation
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol

```solidity
File: /src/EthRouter.sol

91        royaltyRegistry = _royaltyRegistry;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

```solidity
File: /src/PrivatePool.sol

144        factory = payable(_factory);

145        royaltyRegistry = _royaltyRegistry;

146        stolenNftOracle = _stolenNftOracle;

175        baseToken = _baseToken;

176        nft = _nft;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## [G-15] abi.encodePacked() efficient then abi.encode()

### Details

There are **1** instances of this issue.

### Github Permalinks

```solidity
File: /src/PrivatePool.sol

675            leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L675

## [G-16] Access mappings directly rather than using accessor functions

### Details

There are **2** instances of this issue.

### Github Permalinks

```solidity
File: /src/PrivatePool.sol

128        if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {

723        netOutputAmount = outputAmount - feeAmount - protocolFeeAmount;
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol
