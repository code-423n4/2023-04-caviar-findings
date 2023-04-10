### Gas Optimizations List

| Number | Optimization Details | Context |
| --- | --- | --- |
| G-01 | Usage of uint/int smaller than 32 bytes (256 bits) cost more gas | 5   |
| G-02 | `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables | 4   |
| G-03 | `10 ** exponent` can be changed to `1eexponent` and save some gas | 2   |
| G-04 | Use hardcode address instead address(this) | 27  |
| G-05 | Use `selfbalance()` instead of `address(this).balance` | 1 |
|G-06|Using `ternary` operator instead of the `if else` statement saves gas|5|




## \[G-01\] Usage of uints/ints smaller than 32 bytes (256 bits)

State variable declared as uint/int smaller than 32 bytes (256 bits) will cost more gas when used in the code logic as the compiler must first convert them back to uint256 before using them, the only case where using smaller uint/int size matter is when you store many state variables in the same storage slot otherwise it's better to declare them as uint256 to save gas in functions calls.

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L51

```
File: src/Factory.sol
51 uint16 public protocolFeeRate;
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L74-L77

```
File: src/Factory.sol
74       uint128 _virtualBaseTokenReserves,
75       uint128 _virtualNftReserves,
76       uint56 _changeFee,
77       uint16 _feeRate,
```

## \[G-02\] `x += y (x -= y)` costs more gas than `x = x + y (x = x - y)` for state variables

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L231

```
File: src/PrivatePool.sol
231  virtualNftReserves -= uint128(weightSum);
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L323

```
File: src/PrivatePool.sol
323  virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L230

```
File: src/PrivatePool.sol
230 virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L324

```
File: src/PrivatePool.sol
324  virtualNftReserves += uint128(weightSum);
```

## \[G-03\]10 ** 18 can be changed to 1e18 and save some gas

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L734

```diff
File: src/PrivatePool.sol 
-   uint256 feePerNft = changeFee * 10 ** exponent;
+   uint256 feePerNft = changeFee * 1eexponent;
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L745

```diff
FIle: src/PrivatePool.sol
- return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;
+ return (virtualBaseTokenReserves * 1eexponent) / virtualNftReserves;
```

##\[G-04\]Use hardcode address instead address(this).
Instead of address(this), it is more gas-efficient to pre-calculate and use the address with a hardcode. Foundry's script.sol and solmate`LibRlp.sol` contracts can do this.
file:`rc/Factory.sol`
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L169

file:`src/EthRouter.sol`
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L136
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L141
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L142
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L162
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L203
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L208
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L240
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L266
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L285
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L290
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L291

file:`src/PrivatePool.sol`

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L331
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L128
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L240
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L256
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L331
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L423
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L442
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L447
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L497
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L502
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L519
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L638
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L648
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L651
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L766
## [G-05] Use selfbalance() instead of address(this).balance
Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas. Additionally, you can use `
balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L291
```
File:src/EthRouter.so
291 msg.sender.safeTransferETH(address(this).balance);
 ```
##[G-06]Using `ternary` operator instead of the `if else` statement saves gas
File: `src/Factory.sol`
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L110-L116

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L149-L153
File:`src/PrivatePool.sol`
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L278-L282
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L345-L349
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L522-L528