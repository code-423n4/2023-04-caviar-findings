### Gas Optimization Report

## Summary

## Gas Optimizations


|  | Issue | Instances |
| --- | --- | ---|
| 1 | Setting the `constructor` to `payable` | 3 |
| 2 | Functions guaranteed to revert when normal users call can be marked `payable` | 11 |
| 3 | Use Assembly to check for `address(0)` | 21 |
| 4 | Splitting `require` and `if` statements that use `&&` saves gas | 12 |
| 5 | `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables | 4 |


## Gas Optimizations

# [G-01] Setting the `constructor` to `payable`

Setting the `constructor` to `payable` saves about 13 gas per instance

*There are 3 instances of this issue:*

```javascript
File: src/PrivatePool.sol

143    constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle)
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L143


```javascript
File: src/Factory.sol

53    constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L53

```javascript
File: src/EthRouter.sol

90    constructor(address _royaltyRegistry)

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L90

# [G-02] Functions guaranteed to revert when normal users call can be marked `payable`

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are
CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

*There are 11 instances of this issue:*

```javascript
File: src/Factory.sol

129    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner

135    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner 

141    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner
        
148    function withdraw(address token, uint256 amount) public onlyOwner

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L148

```javascript
File: src/PrivatePool.sol

514    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner

538    function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner

550    function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner

562    function setFeeRate(uint16 newFeeRate) public onlyOwner

576    function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner

587    function setPayRoyalties(bool newPayRoyalties) public onlyOwner

602    function setAllParameters
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L514

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L538

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L550

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L562

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L576

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L587

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L602


# [G-03] Use Assembly to check for `address(0)`

Using assembly to check for `address(0)` saves 6 gas per instance.

*There are 21 instances of this issue:*

```javascript
File: src/Factory.sol

87        if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0))

110        if (_baseToken == address(0))

149        if (token == address(0))
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L87

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L110

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L149

```javascript
File: src/PrivatePool.sol

225        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

254        if (baseToken != address(0))

277                if (royaltyFee > 0 && recipient != address(0))
278                    if (baseToken != address(0))

344                if (royaltyFee > 0 && recipient != address(0))
345                    if (baseToken != address(0))

357        if (baseToken == address(0))

397        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

421        if (baseToken != address(0))

489        if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0)))

500        if (baseToken != address(0))

522        if (token == address(0))

635        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

651        if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

733        uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;

744        uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L225

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L254

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L277-L278

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L344-L345

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L357

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L397

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L421

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L489

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L500

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L522

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L635

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L651

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L733

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L744

# [G-04] Splitting `require` and `if` statements that use `&&` saves gas

Instead of using operator `&&` on a single `require` or `if` statement, using two `require` or `if` statements can save 30-70 gas per instance.

*There are 12 instances of this issue:*

```javascript
File: src/Factory.sol

87        if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0))

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L87

```javascript
File: src/EthRouter.sol

101        if (block.timestamp > deadline && deadline != 0)

154        if (block.timestamp > deadline && deadline != 0)

228        if (block.timestamp > deadline && deadline != 0)

256        if (block.timestamp > deadline && deadline != 0)

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L101

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L154

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L228

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L256

```javascript
File: src/PrivatePool.sol

225        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

277                if (royaltyFee > 0 && recipient != address(0))

344                if (royaltyFee > 0 && recipient != address(0))

397        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

489        if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0)))

635        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L225

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L277

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L344

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L397

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L489

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L635

# [G-05] `<x> += <y>` Costs More Gas Than `<x> = <x> + <y>` For State Variables

*There are 4 instances:*

```javascript
File: src/PrivatePool.sol

230        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);

231        virtualNftReserves -= uint128(weightSum);

323        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

324        virtualNftReserves += uint128(weightSum);

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L230-L231

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L323-L324