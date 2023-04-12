## [L-01] changeFee can only be set once & unused parameters in flashFee(address, uint256)
In PrivatePool.sol:
```
87:   /// @notice The change/flash fee to 4 decimals of precision. For example, 0.0025 ETH = 25. 500 USDC = 5_000_000.
88:   uint56 public changeFee;
```
The change fee is then initialized in the initialize(...) function
```
    function initialize(
        address _baseToken,
        address _nft,
        uint128 _virtualBaseTokenReserves,
        uint128 _virtualNftReserves,
        uint56 _changeFee,
        ...
        ...
               changeFee = _changeFee;
```

There are no available setters or ways to change the changeFee (flashFee) after the pool is initialized. Furthermore, flashFee() function takes two parameters that are not used to alter the changeFee in any way.

```
750:    function flashFee(address, uint256) public view returns (uint256) {
            return changeFee;
        }
...
631:   // calculate the fee
632:   uint256 fee = flashFee(token, tokenId);
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L632

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L750



## [I-01] Comment fixes
// add the royalty fee amount to the net input aount 
- fix the "aount" to "amount"
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L251

// loop through and execute the the buys 
- remove "the"

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L105

## [I-02] Confusing/unnecessary imports

Change the import from 
`import {IERC3156FlashBorrower} from         "openzeppelin/interfaces/IERC3156FlashLender.sol";` 
to
```
import {IERC3156FlashBorrower} from  
"openzeppelin/interfaces/IERC3156FlashBorrower.sol";
```
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L34


## [I-03] Remove ReservoirOracle as it is not necessary
The Message struct used in `IStolenNftOracle` is copied from `ReservoirOracle`. Therefore it is not needed to use `ReservoirOracle` at all.
Here the struct of `ReservoirOracle.Message[]` is used but it's better to use the already defined in `IStolenNftOracle`.
```
177:    abi.decode(abi.encode(sells[i].stolenNftProofs), (ReservoirOracle.Message[]))
...

34: import {Pair, ReservoirOracle} from "caviar/Pair.sol";
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L34

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L177


## [I-04] Set functions execute(), setAllParameters() to external in PrivatePool.sol
Functions not used from within the contract can be set to external for code clarity

```
459: function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
...
602: function setAllParameters(
    uint128 newVirtualBaseTokenReserves, ...) public {
```


## [I-05] Set functions setPrivatePoolImplementation(), setProtocolFeeRate(), withdraw(), create(), tokenURI(), predictPoolDeploymentAddress() to external in Factory.sol 
```135:   function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
...
141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
...
148:    function withdraw(address token, uint256 amount) public onlyOwner {
...
71:     function create(...)public payable returns (PrivatePool privatePool)         {
...
161:    function tokenURI(uint256 id) public view override returns (string memory) {
...
168:    function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {

```

## [I-06] Set functions setVirtualReserves(), setMerkleRoot(), setFeeRate(), setUseStolenNftOracle(), setPayRoyalties(), _getRoyalty() to private in PrivatePool.sol

```
538:        function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
...
550:        function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
...
562:        function setFeeRate(uint16 newFeeRate) public onlyOwner {
...
576:        function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
...
587:        function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
...
778:        function _getRoyalty(uint256 tokenId, uint256 salePrice) internal             view returns (uint256 royaltyFee, address recipient) {

```
