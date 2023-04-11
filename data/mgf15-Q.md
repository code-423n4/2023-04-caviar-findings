
## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Constants should be defined rather than using magic numbers | 1 |
| [NC-2](#NC-2) | FLOATING PRAGMAS | 5 |
| [NC-3](#NC-3) | Missing checks for `address(0)` when assigning values to address state variables | 6 |
| [NC-4](#NC-4) | Order of Functions  | * |
| [NC-5](#NC-5) | Some Function Missing events | 3 |
| [NC-6](#NC-6) | Functions guaranteed to revert when called by normal users can be marked payable | 10 |
| [NC-7](#NC-7) | Function Calls in Loop Could Lead to Denial of Service | 19 |

### [NC-1] Constants should be defined rather than using magic numbers

*Instances (1)*:
```solidity
File: 2023-04-caviar/src/PrivatePool.sol

744:         uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());

```
### [NC-2] FLOATING PRAGMAS

It is a best practice to lock pragmas instead of using floating pragmas to ensure that contracts are tested and deployed with the intended compiler version. Accidentally deploying contracts with different compiler versions can lead to unexpected risks and undiscovered bugs. Please consider locking pragmas for the following files.

*Instances (5)*:
```solidity
File: 2023-04-caviar/blob/main/src/Factory.sol

2: pragma solidity ^0.8.19;
```

```solidity
File: 2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol

2: pragma solidity ^0.8.19;
```

```solidity
File: 2023-04-caviar/blob/main/src/EthRouter.sol

2: pragma solidity ^0.8.19;
```

```solidity
File: 2023-04-caviar/blob/main/src/PrivatePool.sol

2: pragma solidity ^0.8.19;
```

### [NC-3] Missing checks for `address(0)` when assigning values to address state variables 

Zero-address check should be used in the constructors, to avoid the risk of setting a storage variable as address(0) at deploying time.

*Instances (6)*:
```solidity
File: 2023-04-caviar/blob/main/src/EthRouter.sol

90:   constructor(address _royaltyRegistry) {
91:        royaltyRegistry = _royaltyRegistry;
92:    }
```

```solidity
File: 2023-04-caviar/blob/main/src/PrivatePool.sol

143:   constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
144:        factory = payable(_factory);
145:        royaltyRegistry = _royaltyRegistry;
146:        stolenNftOracle = _stolenNftOracle;
147:    }

175:        baseToken = _baseToken;
176:        nft = _nft;
```

### [NC-4] Order of function 

Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.

Functions should be grouped according to their visibility and ordered:

* constructor

* receive function (if exists)

* fallback function (if exists)

* external

* public

* internal

* private

```
File : 2023-04-caviar/blob/main/src/PrivatePool.sol
```
ref 

https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions


### [NC-5] Some function mess events 

```solidity
File: 2023-04-caviar/blob/main/src/Factory.sol

129:    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
130:        privatePoolMetadata = _privatePoolMetadata;
131:    }

135:    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
136:        privatePoolImplementation = _privatePoolImplementation;
137:    }

141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
142:        protocolFeeRate = _protocolFeeRate;
143:    }

```

### [NC-6] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

*Instances (10)*:
```solidity
File: 2023-04-caviar/blob/main/src/Factory.sol

129:    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:    function withdraw(address token, uint256 amount) public onlyOwner {
```

```solidity
File: 2023-04-caviar/blob/main/src/PrivatePool.sol


514:    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

538:    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {

550:    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {

562:    function setFeeRate(uint16 newFeeRate) public onlyOwner {

576:    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {

578:    function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```

### [NC-7] Function Calls in Loop Could Lead to Denial of Service

Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to the gas limitations or failed transactions. Here are some of the instances entailed:


*Instances (19)*:
```solidity
File: 2023-04-caviar/src/EthRouter.sol

106:         for (uint256 i = 0; i < buys.length; i++) {

116:                     for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

134:             for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {

159:         for (uint256 i = 0; i < sells.length; i++) {

161:             for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

183:                     for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {

239:         for (uint256 i = 0; i < tokenIds.length; i++) {

261:         for (uint256 i = 0; i < changes.length; i++) {

265:             for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {

284:             for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {

```

```solidity
File: 2023-04-caviar/src/Factory.sol

119:         for (uint256 i = 0; i < tokenIds.length; i++) {

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

238:         for (uint256 i = 0; i < tokenIds.length; i++) {

272:             for (uint256 i = 0; i < tokenIds.length; i++) {

329:         for (uint256 i = 0; i < tokenIds.length; i++) {

441:         for (uint256 i = 0; i < inputTokenIds.length; i++) {

446:         for (uint256 i = 0; i < outputTokenIds.length; i++) {

496:         for (uint256 i = 0; i < tokenIds.length; i++) {

518:         for (uint256 i = 0; i < tokenIds.length; i++) {

673:         for (uint256 i = 0; i < tokenIds.length; i++) {

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Initializers could be front-run | 2 |
| [L-2](#L-2) | Unsafe ERC20 operation(s) | 5 |
| [L-3](#L-3) | Unused/empty receive()/fallback() function | 3 |

### [L-1] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (2)*:
```solidity
File: 2023-04-caviar/src/Factory.sol

98:         privatePool.initialize(

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

157:     function initialize(

```

### [L-2] Unsafe ERC20 operation(s)

*Instances (5)*:
```solidity
File: 2023-04-caviar/src/Factory.sol

115:             ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);

152:             ERC20(token).transfer(msg.sender, amount);

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

365:             ERC20(baseToken).transfer(msg.sender, netOutputAmount);

527:             ERC20(token).transfer(msg.sender, tokenAmount);

651:         if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

```

### [L‑03] Unused/empty receive()/fallback() function 

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth))). Having no access control on the function means that someone may send Ether to the contract, and have no way to get anything back out, which is a loss of funds

*Instances (3)*:

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

134:    receive() external payable {}
```
```solidity
File: 2023-04-caviar/src/Factory.sol

55:    receive() external payable {}
```

```solidity
File: 2023-04-caviar/src/EthRouter.sol

88:    receive() external payable {}

```