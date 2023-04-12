# Gas Optimizations list

| Number | Details                                                                                           | Instances |
| ------ | ------------------------------------------------------------------------------------------------- | --------- |
| 1      | ```x += y```/```x -= y``` COSTS MORE GAS THAN ```x = x + y```/```x = x - y``` FOR STATE VARIABLES | 4        |
| 2      | CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS                                                     | 13         |
| 3      | PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD                   | 21         |
| 4      | OPTIMIZE NAMES TO SAVE GAS                                                                        | 4        |
| 5     | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE                       | 1         |
| 6     | REORDER STRUCTURE LAYOUT                                                                          | 2         |
| 7     | USE ASSEMBLY TO CHECK FOR ```address(0)```                                                        | 21        |

# Gas Optimizations
## [G-01]  ```x += y```/```x -= y``` COSTS MORE GAS THAN ```x = x + y```/```x = x - y``` FOR STATE VARIABLES
Using the addition operator instead of plus-equals saves some gas for state variables.

src\PrivatePool.sol
```js
230:         virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231:         virtualNftReserves -= uint128(weightSum);

323:         virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324:         virtualNftReserves += uint128(weightSum);
```

## [G-02] CAN DECLARE VARIABLE OUTSIDE LOOP TO SAVE GAS 
Declaring the stack variable outside the loop will save gas that would otherwise be spent on declaring the variable over and over again.     

src\EthRouter.sol
```js
109:                 uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(

115:                     uint256 salePrice = inputAmount / buys[i].tokenIds.length;

118:                         (uint256 royaltyFee, address royaltyRecipient) =

170:                 uint256 outputAmount = Pair(sells[i].pool).nftSell(

182:                     uint256 salePrice = outputAmount / sells[i].tokenIds.length;

185:                         (uint256 royaltyFee, address royaltyRecipient) =

262:             Change memory _change = changes[i];
```

src\PrivatePool.sol
```js
274:                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);

335:                 uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;

338:                 (uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);
```

## [G-03] PUBLIC FUNCTIONS NOT CALLED BY THE CONTRACT SHOULD BE DECLARED EXTERNAL INSTEAD
Contracts are allowed to override their parents’ functions and change the visibility from public to external and can save gas by doing so.

src\EthRouter.sol
```js
99:     function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {

219:     function deposit(
220:         address payable privatePool,
221:         address nft,
222:         uint256[] calldata tokenIds,
223:         uint256 minPrice,
224:         uint256 maxPrice,
225:         uint256 deadline
226:     ) public payable {

254:     function change(Change[] calldata changes, uint256 deadline) public payable {
```

src\Factory.sol
```js
71:     function create(
72:         address _baseToken,
73:         address _nft,
74:         uint128 _virtualBaseTokenReserves,
75:         uint128 _virtualNftReserves,
76:         uint56 _changeFee,
77:         uint16 _feeRate,
78:         bytes32 _merkleRoot,
79:         bool _useStolenNftOracle,
80:         bool _payRoyalties,
81:         bytes32 _salt,
82:         uint256[] memory tokenIds, // put in memory to avoid stack too deep error
83:         uint256 baseTokenAmount
84:     ) public payable returns (PrivatePool privatePool) {

129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {

135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {

141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

148:     function withdraw(address token, uint256 amount) public onlyOwner {

168:     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {
```

src\PrivatePool.sol
```js
157:     function initialize(
158:         address _baseToken,
159:         address _nft,
160:         uint128 _virtualBaseTokenReserves,
161:         uint128 _virtualNftReserves,
162:         uint56 _changeFee,
163:         uint16 _feeRate,
164:         bytes32 _merkleRoot,
165:         bool _useStolenNftOracle,
166:         bool _payRoyalties
167:     ) public {

211:     function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
212:         public
213:         payable
214:         returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)

301:     function sell(
302:         uint256[] calldata tokenIds,
303:         uint256[] calldata tokenWeights,
304:         MerkleMultiProof calldata proof,
305:         IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
306:     ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {

385:     function change(
386:         uint256[] memory inputTokenIds,
387:         uint256[] memory inputTokenWeights,
388:         MerkleMultiProof memory inputProof,
389:         IStolenNftOracle.Message[] memory stolenNftProofs,
390:         uint256[] memory outputTokenIds,
391:         uint256[] memory outputTokenWeights,
392:         MerkleMultiProof memory outputProof
393:     ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {

459:     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {

484:     function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {

514:     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {

602:     function setAllParameters(
603:         uint128 newVirtualBaseTokenReserves,
604:         uint128 newVirtualNftReserves,
605:         bytes32 newMerkleRoot,
606:         uint16 newFeeRate,
607:         bool newUseStolenNftOracle,
608:         bool newPayRoyalties
609:     ) public {

742:     function price() public view returns (uint256) {
```

src\PrivatePoolMetadata.sol
```js
17:     function tokenURI(uint256 tokenId) public view returns (string memory) {
```

## [G-04] OPTIMIZE NAMES TO SAVE GAS
Function hashes that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted. Prioritize the most called functions and sort and rename them according to the function hashes/method IDs. For a better understanding please refer to [this link](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92)

The method IDs in the PrivatePool.sol will be used the most. A lower method ID may be given to the most frequently used functions. [This](https://emn178.github.io/solidity-optimize-name/) is a useful tool that can be used for the same, if required.

```json
Sighash   |   Function Signature
========================
7c5fc1ea  =>  initialize(address,address,uint128,uint128,uint56,uint16,bytes32,bool,bool)
2e02d585  =>  buy(uint256[],uint256[],MerkleMultiProof)
57cb36d4  =>  sell(uint256[],uint256[],MerkleMultiProof,IStolenNftOracle.Message[])
f77c16e1  =>  change(uint256[],uint256[],MerkleMultiProof,IStolenNftOracle.Message[],uint256[],uint256[],MerkleMultiProof)
1cff79cd  =>  execute(address,bytes)
5dc55f2f  =>  deposit(uint256[],uint256)
6de85209  =>  withdraw(address,uint256[],address,uint256)
f848095d  =>  setVirtualReserves(uint128,uint128)
7cb64759  =>  setMerkleRoot(bytes32)
29ff0773  =>  setFeeRate(uint16)
42973a53  =>  setUseStolenNftOracle(bool)
58785f6f  =>  setPayRoyalties(bool)
85aa8356  =>  setAllParameters(uint128,uint128,bytes32,uint16,bool,bool)
d3f2b14a  =>  flashLoan(IERC3156FlashBorrower,address,uint256,bytes)
81b401c1  =>  sumWeightsAndValidateProof(uint256[],uint256[],MerkleMultiProof)
168585e5  =>  buyQuote(uint256)
d7a2e4c9  =>  sellQuote(uint256)
f6fb3f8f  =>  changeFeeQuote(uint256)
a035b1fe  =>  price()
d9d98ce4  =>  flashFee(address,uint256)
e3afca5c  =>  flashFeeToken()
474d22a4  =>  availableForFlashLoan(address,uint256)
b546f8d5  =>  _getRoyalty(uint256,uint256)
```

## [G-05] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE
Before transfer, we should check for amount being 0 so the function doesnt run when its not gonna do anything.

src\PrivatePool.sol
```js
527:             ERC20(token).transfer(msg.sender, tokenAmount);
```

## [G-06] REORDER STRUCTURE LAYOUT
The following structs could be optimized by moving the position of certain variables in order to optimize storage.

src\EthRouter.sol
```js
48:     struct Buy {
49:         address payable pool;
50:         address nft;
51:         uint256[] tokenIds;
52:         uint256[] tokenWeights;
53:         PrivatePool.MerkleMultiProof proof;
54:         uint256 baseTokenAmount;
55:         bool isPublicPool;
56:     }

58:     struct Sell {
59:         address payable pool;
60:         address nft;
61:         uint256[] tokenIds;
62:         uint256[] tokenWeights;
63:         PrivatePool.MerkleMultiProof proof;
64:         IStolenNftOracle.Message[] stolenNftProofs;
65:         bool isPublicPool;
66:         bytes32[][] publicPoolProofs;
67:     }
```

## [G-07] USE ASSEMBLY TO CHECK FOR ```address(0)```
Saves 6 gas per instance if using assembly to check for address(0).

src\Factory.sol
```js
87:         if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {

110:         if (_baseToken == address(0)) {

149:         if (token == address(0)) {
```

src\PrivatePool.sol
```js
225:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

254:         if (baseToken != address(0)) {

277:                 if (royaltyFee > 0 && recipient != address(0)) {
  
278:                     if (baseToken != address(0)) {

344:                 if (royaltyFee > 0 && recipient != address(0)) {
  
345:                     if (baseToken != address(0)) {

357:         if (baseToken == address(0)) {

397:         if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

421:         if (baseToken != address(0)) {

489:         if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

500:         if (baseToken != address(0)) {

522:         if (token == address(0)) {

635:         if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

651:         if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

733:         uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;

744:         uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
```

src\PrivatePoolMetadata.sol
```js
47:             trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(addres(privatePool))))


103:                         "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceO(address(privatePool))),
```