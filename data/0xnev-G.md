### Gas Optimizations
| |Issue|Instances|Total Gas Saved|
|:-:|:-|:-:|:-:|
| [G-01] |  `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (same for -=) | 4 |  452 |
| [G-02] | Use nested `if` statements to avoid multiple check combinations using `&&` | 9 |  54 |
| [G-03] | Structs can be packed in to fewer storage slots | 2 |  40000 |
| [G-04] | Public functions not called by contract can be declared external instead | 6 |  - |
| [G-05] | Functions guaranteed to revert when called by normal users can be marked payable | 10 | 210 |
| [G-06] | Refactor `PrivatePool.availableForFlashLoan()` | 1 |  2236 |
| [G-07] | `PrivatePool.flashFee()` function not required since it simply returns `changeFee` | 1 | 88510 |
| [G-08] | `PrivatePool.flashFeeToken()` function not required  | 1 | 46871 |

| Total Found Issues | 8 |
|:--:|:--:|
| Total Gas Savings | Estimated >= 178333 |


### [G-01] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables (same for -=)
[PrivatePool.sol#L230-L231](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230-L231)
[PrivatePool.sol#L323-L324](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323-L324)
```solidity
4 result - 1 file

/PrivatePool.sol
230:        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231:        virtualNftReserves -= uint128(weightSum);

323:        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324:        virtualNftReserves += uint128(weightSum);
```
Using the addition operator instead of plus-equals saves 113 gas. This applies to `-=` too.

### [G-02] Use nested `if` statements to avoid multiple check combinations using `&&`
[EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101)
[EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154)
[EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228)
[EthRouter.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256)
```solidity
9 results - 2 files

/EthRouter.sol
101:        if (block.timestamp > deadline && deadline != 0)

154:        if (block.timestamp > deadline && deadline != 0)

228:        if (block.timestamp > deadline && deadline != 0)

256:        if (block.timestamp > deadline && deadline != 0) 
```
[PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225)
[PrivatePool.sol#L277](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277)
[PrivatePool.sol#L344](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344)
[PrivatePool.sol#L397](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397)
[PrivatePool.sol#L635](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635)
```solidity
/PrivatePool.sol
225:        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

277:                if (royaltyFee > 0 && recipient != address(0))

344:                if (royaltyFee > 0 && recipient != address(0))

397:        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

635:        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
```

Using nested `if` statements is cheaper than using `&& ` for multiple check combinations (saves ~6gas). Additionally, it improves readability of code and better coverage reports.

### [G-03] Structs can be packed in to fewer storage slots
[EthRouter.sol#L55](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L55)
```solidity
2 results - 1 file

/EthRouter.sol
48:    struct Buy {
49:        address payable pool;
50:        address nft;
51:        uint256[] tokenIds;
52:        uint256[] tokenWeights;
53:        PrivatePool.MerkleMultiProof proof;
54:        uint256 baseTokenAmount;
55:        bool isPublicPool;
56:    }
```
[EthRouter.sol#L65](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L65)
```solidity
58:    struct Sell {
59:        address payable pool;
60:        address nft;
61:        uint256[] tokenIds;
62:        uint256[] tokenWeights;
63:        PrivatePool.MerkleMultiProof proof;
64:        IStolenNftOracle.Message[] stolenNftProofs;
65:        bool isPublicPool;
66:        bytes32[][] publicPoolProofs;
67:    }
```

Adhere to storage packing rules to avoid an extra Gsset (20000gas) for the first setting of the struct. Subsequent reads are also cheaper.

Change to:
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
```

```solidity
struct Sell {
    address payable pool;
    address nft;
    uint256[] tokenIds;
    uint256[] tokenWeights;
    bool isPublicPool;
    PrivatePool.MerkleMultiProof proof;
    IStolenNftOracle.Message[] stolenNftProofs;
    bytes32[][] publicPoolProofs;
}
```

### [G-04] Public functions not called by contract can be declared external instead
[Factory.sol#L71](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L71)
[Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148)
```solidity
6 results - 2 files

/Factory.sol
71:    function create(
72:        address _baseToken,
73:        address _nft,
74:        uint128 _virtualBaseTokenReserves,
75:        uint128 _virtualNftReserves,
76:        uint56 _changeFee,
77:        uint16 _feeRate,
78:        bytes32 _merkleRoot,
79:        bool _useStolenNftOracle,
80:        bool _payRoyalties,
81:        bytes32 _salt,
82:        uint256[] memory tokenIds, // put in memory to avoid stack too deep error
83:        uint256 baseTokenAmount
84:    ) public payable returns (PrivatePool privatePool)

148:    function withdraw(address token, uint256 amount) public onlyOwner
```
[EthRouter.sol#L99](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L99)
[EthRouter.sol#L152](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L152)
[EthRouter.sol#L219](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L219)
[EthRouter.sol#L254](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254)
```solidity
/EthRouter.sol
99:    function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable 

152:    function sell(Sell[] calldata sells, uint256 minOutputAmount, uint256 deadline, bool payRoyalties) public

219:    function deposit(
220:        address payable privatePool,
221:        address nft,
222:        uint256[] calldata tokenIds,
223:        uint256 minPrice,
224:        uint256 maxPrice,
225:        uint256 deadline
226:    ) public payable

254:    function change(Change[] calldata changes, uint256 deadline) public payable
```
The above functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

### [G-05] Functions guaranteed to revert when called by normal users can be marked payable
[Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129)
[Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135)
[Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141)
[Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148)
```solidity
10 results - 2 files

/Factory.sol
129:    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner

135:    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner

141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner

148:    function withdraw(address token, uint256 amount) public onlyOwner
```
[PrivatePool.sol#L514](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514)
[PrivatePool.sol#L538](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538)
[PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550)
[PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562)
[PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576)
[PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587)
```solidity
/PrivatePool.sol
514:    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner 

538:    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner

550:    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner

562:    function setFeeRate(uint16 newFeeRate) public onlyOwner

576:    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner

587:    function setPayRoyalties(bool newPayRoyalties) public onlyOwner
```

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

### [G-06] Refactor `PrivatePool.availableForFlashLoan()`
[PrivatePool.sol#L763-L769](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L763-L769)
```solidity
/PrivatePool.sol
763:    function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {
764:        // return if the NFT is owned by this contract
765:        try ERC721(token).ownerOf(tokenId) returns (address result) {
766:            return result == address(this);
767:        } catch {
768:            return false;
769:        }
```

The function `PrivatePool.availableForFlashLoan()` can be refactored to simply checking `ERC721(token).ownerOf(tokenId) == address(this);`

```solidity
function availableForFlashLoan(address token, uint256 tokenId) public view returns (bool) {
    // return if the NFT is owned by this contract
    return ERC721(token).ownerOf(tokenId) == address(this);
}
```

### [G-07] `PrivatePool.flashFee()` function not required since it simply returns `changeFee`
[PrivatePool.sol#L750-L752](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L750-L752)
```solidity
/PrivatePool.sol
750:    function flashFee(address, uint256) public view returns (uint256) {
751:        return changeFee;
752:    }
```

Since `PrivatePool.flashFee()`  always returns `changeFee`, functions using  `PrivatePool.flashFee()` can simply use `changeFee` to avoid a external call (saves ~20 gas by avoiding JUMP). 

Furthermore, since PrivatePool.sol is not inherited, there is no child contracts to change value of `changeFee` with `PrivatePool.flashFee()`. if user wants to view the value of `changeFee`, solidity already has an automatic getter function to access the value. As such, we can remove the `PrivatePool.flashFee()` function.

### [G-08] `PrivatePool.flashFeeToken()` function not required 
[PrivatePool.sol#L755-L757](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L755-L757)
```solidity
/PrivatePool.sol
755:    function flashFeeToken() public view returns (address) {
756:        return baseToken;
757:    }
```

Similar to [g], since PrivatePool.sol is not inherited, there is no child contracts to change value of `baseToken` with `PrivatePool.flashFeeToken()`. if user wants to view the address value of `baseToken`, solidity already has an automatic getter function to access the value. As such, we can remove the `PrivatePool.flashFeeToken()` function.