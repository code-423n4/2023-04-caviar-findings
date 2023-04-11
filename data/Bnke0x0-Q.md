

### [L01] Missing checks for `address(0x0)` when assigning values to `address` state variables


#### Findings:
```
2023-04-caviar/src/EthRouter.sol::91 => royaltyRegistry = _royaltyRegistry;
2023-04-caviar/src/Factory.sol::130 => privatePoolMetadata = _privatePoolMetadata;
2023-04-caviar/src/Factory.sol::136 => privatePoolImplementation = _privatePoolImplementation;
2023-04-caviar/src/Factory.sol::142 => protocolFeeRate = _protocolFeeRate;
2023-04-caviar/src/PrivatePool.sol::145 => royaltyRegistry = _royaltyRegistry;
2023-04-caviar/src/PrivatePool.sol::146 => stolenNftOracle = _stolenNftOracle;
2023-04-caviar/src/PrivatePool.sol::175 => baseToken = _baseToken;
2023-04-caviar/src/PrivatePool.sol::176 => nft = _nft;
2023-04-caviar/src/PrivatePool.sol::177 => virtualBaseTokenReserves = _virtualBaseTokenReserves;
2023-04-caviar/src/PrivatePool.sol::178 => virtualNftReserves = _virtualNftReserves;
2023-04-caviar/src/PrivatePool.sol::179 => changeFee = _changeFee;
2023-04-caviar/src/PrivatePool.sol::180 => feeRate = _feeRate;
2023-04-caviar/src/PrivatePool.sol::181 => merkleRoot = _merkleRoot;
2023-04-caviar/src/PrivatePool.sol::182 => useStolenNftOracle = _useStolenNftOracle;
2023-04-caviar/src/PrivatePool.sol::183 => payRoyalties = _payRoyalties;
```


### [L02] Unused `receive()` function will lock Ether in contract

#### Impact
If the intention is for the Ether to be used, the function should call another function, otherwise, it should revert
#### Findings:
```
2023-04-caviar/src/EthRouter.sol::88 => receive() external payable {}
2023-04-caviar/src/Factory.sol::55 => receive() external payable {}
2023-04-caviar/src/PrivatePool.sol::134 => receive() external payable {}
```





### [L03] Unspecific Compiler Version Pragma

#### Impact
For most source-units the compiler version pragma is very unspecific `^0.8.19`. While this often makes sense for libraries to allow them to be included with multiple different versions of an application, it may be a security risk for the actual application implementation itself. A known vulnerable compiler version may accidentally be selected or security tools might fall-back to and older compiler version ending up actually checking a different evm compilation that is ultimately deployed on the blockchain.
#### Findings:
```
2023-04-caviar/src/EthRouter.sol::2 => pragma solidity ^0.8.19;
2023-04-caviar/src/Factory.sol::2 => pragma solidity ^0.8.19;
2023-04-caviar/src/PrivatePool.sol::2 => pragma solidity ^0.8.19;
2023-04-caviar/src/PrivatePoolMetadata.sol::2 => pragma solidity ^0.8.19;
2023-04-caviar/src/interfaces/IStolenNftOracle.sol::2 => pragma solidity ^0.8.19;
```


### Non-Critical Issues



### [N01] Adding a return statement when the function defines a named return variable, is redundant


#### Findings:
```
2023-04-caviar/src/PrivatePool.sol::686 => return sum;
2023-04-caviar/src/PrivatePool.sol::751 => return changeFee;
2023-04-caviar/src/PrivatePool.sol::756 => return baseToken;
2023-04-caviar/src/PrivatePoolMetadata.sol::109 => return _svg;
```
#### Tools used
[c4udit](https://github.com/Bnke0x0/c4udit)

### [N02] `require()`/`revert()` statements should have descriptive reason strings


#### Findings:
```
2023-04-caviar/src/PrivatePool.sol::471 => revert(add(32, returnData), returnData_size)
2023-04-caviar/src/PrivatePool.sol::474 => revert();
```





### [N03] Event is missing `indexed` fields

#### Impact
Each event should use three indexed fields if there are three or more fields
#### Findings:
```
2023-04-caviar/src/Factory.sol::41 => event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);
2023-04-caviar/src/PrivatePool.sol::61 => event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);
2023-04-caviar/src/PrivatePool.sol::62 => event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);
2023-04-caviar/src/PrivatePool.sol::63 => event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
2023-04-caviar/src/PrivatePool.sol::64 => event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);
```


### [N04] Unused file


#### Findings:
```
2023-04-caviar/src/EthRouter.sol::1 => // SPDX-License-Identifier: MIT
2023-04-caviar/src/Factory.sol::1 => // SPDX-License-Identifier: MIT
2023-04-caviar/src/PrivatePool.sol::1 => // SPDX-License-Identifier: MIT
2023-04-caviar/src/PrivatePoolMetadata.sol::1 => // SPDX-License-Identifier: MIT
2023-04-caviar/src/interfaces/IStolenNftOracle.sol::1 => // SPDX-License-Identifier: MIT
```



### [N05] `public` functions not called by the contract should be declared `external` instead

#### Impact
Contracts are allowed to override their parents’ functions and change the visibility from public to external .
#### Findings:
```
2023-04-caviar/src/EthRouter.sol::99 => function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
2023-04-caviar/src/EthRouter.sol::152 => function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public {
2023-04-caviar/src/EthRouter.sol::254 => function change(Change[] calldata changes, uint256 deadline) public payable {
2023-04-caviar/src/Factory.sol::129 => function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
2023-04-caviar/src/Factory.sol::135 => function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
2023-04-caviar/src/Factory.sol::141 => function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
2023-04-caviar/src/Factory.sol::148 => function withdraw(address token, uint256 amount) public onlyOwner {
2023-04-caviar/src/Factory.sol::161 => function tokenURI(uint256 id) public view override returns (string memory) {
2023-04-caviar/src/Factory.sol::168 => function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {
2023-04-caviar/src/PrivatePool.sol::459 => function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
2023-04-caviar/src/PrivatePool.sol::484 => function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {
2023-04-caviar/src/PrivatePool.sol::514 => function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::538 => function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::550 => function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::562 => function setFeeRate(uint16 newFeeRate) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::576 => function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::587 => function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
2023-04-caviar/src/PrivatePool.sol::731 => function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
2023-04-caviar/src/PrivatePool.sol::742 => function price() public view returns (uint256) {
2023-04-caviar/src/PrivatePool.sol::750 => function flashFee(address, uint256) public view returns (uint256) {
2023-04-caviar/src/PrivatePool.sol::755 => function flashFeeToken() public view returns (address) {
2023-04-caviar/src/PrivatePool.sol::763 => function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {
2023-04-caviar/src/PrivatePoolMetadata.sol::17 => function tokenURI(uint256 tokenId) public view returns (string memory) {
2023-04-caviar/src/PrivatePoolMetadata.sol::35 => function attributes(uint256 tokenId) public view returns (string memory) {
2023-04-caviar/src/PrivatePoolMetadata.sol::55 => function svg(uint256 tokenId) public view returns (bytes memory) {
```



### [N06] NC-library/interface files should use fixed compiler versions, not floating ones


#### Findings:
```
2023-04-caviar/src/EthRouter.sol::2 => pragma solidity ^0.8.19;
2023-04-caviar/src/Factory.sol::2 => pragma solidity ^0.8.19;
2023-04-caviar/src/PrivatePool.sol::2 => pragma solidity ^0.8.19;
2023-04-caviar/src/PrivatePoolMetadata.sol::2 => pragma solidity ^0.8.19;
2023-04-caviar/src/interfaces/IStolenNftOracle.sol::2 => pragma solidity ^0.8.19;
```



