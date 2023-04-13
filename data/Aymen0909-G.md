# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1  | `storage` variable should be cached into `memory` variables instead of re-reading them  |  7 |
| 2  | `salePrice` should be calculated outside the for loop |  1 |
| 3  | Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead  |  5 |
| 4  | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables  |  4 |
| 5  | Input check statements should be placed at the start of the functions |  2 |
| 6  | Duplicated input check statements should be refactored to a modifier | 4 |
| 7  | `public` functions not called by the contract should be declared `external` instead | 20 |


## Findings

### 1- `storage` variable should be cached into `memory` variables instead of re-reading them :

The instances below point to the second+ access of a state variable within a function, the caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read, thus saves **100gas** for each instance.

There are 7 instances of this issue :

File: PrivatePool.sol 

[Line 225-279](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225-L279)

In the code linked above the value of `baseToken` is read multiple times (6) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 345-368](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345-L368)

In the code linked above the value of `baseToken` is read multiple times (5) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 397-426](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397-L426)

In the code linked above the value of `baseToken` is read multiple times (4) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 401-447](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L401-L447)

In the code linked above the value of `nft` is read multiple times (3) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 489-502](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489-L502)

In the code linked above the value of `baseToken` is read multiple times (4) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 635-651](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635-L651)

In the code linked above the value of `baseToken` is read multiple times (3) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

[Line 667-682](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L667-L682)

In the code linked above the value of `merkleRoot` is read multiple times (2) from storage and it's value does not change so it should be cached into a memory variable in order to save gas by avoiding multiple reads from storage.

### 2- `salePrice` should be calculated outside the for loop :

In the `sell` function from the PrivatePool contract, the `salePrice` value should be calculated outside of the for loop to save gas, as its value will remain the same throughout the iterations.

There is 1 instances of this issue:

File: PrivatePool.sol [Line 335](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335)
```
uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;
```

### 3- Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead (saves ~1127 gas):

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size as you can check [here](https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html).

So use `uint256`/`int256` for state variables and then downcast to lower sizes where needed.

There are 5 instance of this issue:

File: Factory.sol [Line 51](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L51)
```
uint16 public protocolFeeRate;
```

File: Factory.sol 

[Line 88](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L88)
```
uint56 public changeFee;
```

[Line 91](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L91)
```
uint16 public feeRate;
```

[Line 104](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L104)
```
uint128 public virtualBaseTokenReserves;
```

[Line 112](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L112)
```
uint128 public virtualNftReserves;
```

### 4- `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables :

Using the addition operator instead of plus-equals saves **113 gas** as explained [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 4 instances of this issue:

```
File: PrivatePool.sol

230     virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
231     virtualNftReserves -= uint128(weightSum);
323     virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
324     virtualNftReserves += uint128(weightSum);
```


### 5- Input check statements should be placed at the start of the functions :

The check statements on the functions input values should be placed at the beginning of the functions to avoid stack too deep errors and save gas.

There are 2 instances of this issue:

File: PrivatePool.sol [Line 225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225)
```
if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
```

File: PrivatePool.sol [Line 316-318](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L316-L318)
```
if (useStolenNftOracle) {
    IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
}
```

### 6- Duplicated input check statements should be refactored to a modifier :

The following check statements are repeated multiple times (4) :

```
File: EthRouter.sol

101     if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
154     if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
228     if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
256     if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
```

Because this check is done at the start of the function it can be refactored into a modifier to save gas, it should be replaced by a `validDeadline` modifier as follow :

```
modifier validDeadline(uint256 deadline){
    if (block.timestamp > deadline && deadline != 0) {
        revert DeadlinePassed();
    }
    _;
}
```


### 7- `public` functions not called by the contract should be declared `external` instead :

The `public` functions that are not called inside the contract should be declared `external` instead to save gas.

There are 20 instances of this issue:

```
File: PrivatePool.sol

211     function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof) public
301     function sell(
            uint256[] calldata tokenIds,
            uint256[] calldata tokenWeights,
            MerkleMultiProof calldata proof,
            IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
        ) public
385     function change(
            uint256[] memory inputTokenIds,
            uint256[] memory inputTokenWeights,
            MerkleMultiProof memory inputProof,
            IStolenNftOracle.Message[] memory stolenNftProofs,
            uint256[] memory outputTokenIds,
            uint256[] memory outputTokenWeights,
            MerkleMultiProof memory outputProof
        ) public
459     function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory)
484     function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable
514     function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public
602     function setAllParameters(
            uint128 newVirtualBaseTokenReserves,
            uint128 newVirtualNftReserves,
            bytes32 newMerkleRoot,
            uint16 newFeeRate,
            bool newUseStolenNftOracle,
            bool newPayRoyalties
        ) public
742     function price() public view returns (uint256)
755     function flashFeeToken() public view returns (address)

File: PrivatePoolMetadata.sol

17  	function tokenURI(uint256 tokenId) public view returns (string memory)

File: EthRouter.sol

99      function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public
152     function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public
219     function deposit(
            address payable privatePool,
            address nft,
            uint256[] calldata tokenIds,
            uint256 minPrice,
            uint256 maxPrice,
            uint256 deadline
        ) public
254     function change(Change[] calldata changes, uint256 deadline) public

File: Factory.sol

129     function setPrivatePoolMetadata(address _privatePoolMetadata) public
135     function setPrivatePoolImplementation(address _privatePoolImplementation) public 
141     function setProtocolFeeRate(uint16 _protocolFeeRate) public
148     function withdraw(address token, uint256 amount) public 
161     function tokenURI(uint256 id) public view override returns (string memory) 
168     function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress)
```