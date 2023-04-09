### Gas Optimizations List

| Number | Optimization Details                                                               | Instances |
| :----: | :--------------------------------------------------------------------------------- | :-------: |
| [G-01] | Mark functions as payable (with discretion)                                        |    28     |
| [G-02] | Use assembly to check for address(0)                                               |    21     |
| [G-03] | Right shift or Left shift instead of dividing or multiplying by powers of two      |     2     |
| [G-04] | Use `calldata` instead of `memory` for function arguments that do not get mutated. |    12     |
| [G-05] | Use assembly to write storage values                                               |    19     |
| [G-06] | Use assembly when getting a contract's balance of ETH.                             |     8     |
| [G-07] | Pack structs                                                                       |     2     |
| [G-08] | Use assembly to hash instead of Solidity                                           |     2     |
| [G-09] | Use assembly for math (add, sub, mul, div)                                         |    24     |

Total 9 issues

## [G-01] Mark functions as payable (with discretion)

You can mark public or external functions as payable to save gas. Functions that are not payable have additional logic to check if there was a value sent with a call, however, making a function payable eliminates this check. This optimization should be carefully considered due to potentially unwanted behavior when a function does not need to accept ether.

### Lines

- [PrivatePool.sol:157](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157)
- [PrivatePool.sol:301](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301)
- [PrivatePool.sol:514](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514)
- [PrivatePool.sol:538](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538)
- [PrivatePool.sol:550](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550)
- [PrivatePool.sol:562](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562)
- [PrivatePool.sol:576](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576)
- [PrivatePool.sol:587](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587)
- [PrivatePool.sol:602](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L602)
- [PrivatePool.sol:661](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L661)
- [PrivatePool.sol:694](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L694)
- [PrivatePool.sol:713](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L713)
- [PrivatePool.sol:731](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L731)
- [PrivatePool.sol:742](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L742)
- [PrivatePool.sol:750](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L750)
- [PrivatePool.sol:755](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L755)
- [PrivatePool.sol:763](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L763)
- [Factory.sol:129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129)
- [Factory.sol:135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135)
- [Factory.sol:141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141)
- [Factory.sol:148](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148)
- [Factory.sol:161](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L161)
- [Factory.sol:168](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L168)
- [PrivatePoolMetadata.sol:17](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17)
- [PrivatePoolMetadata.sol:35](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L35)
- [PrivatePoolMetadata.sol:55](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L55)
- [EthRouter.sol:152](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L152)
- [EthRouter.sol:301](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L301)

## [G-02] Use assembly to check for address(0)

### Lines

- [PrivatePool.sol:225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225)
- [PrivatePool.sol:254](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L254)
- [PrivatePool.sol:277](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277)
- [PrivatePool.sol:278](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L278)
- [PrivatePool.sol:344](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344)
- [PrivatePool.sol:345](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345)
- [PrivatePool.sol:357](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L357)
- [PrivatePool.sol:397](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397)
- [PrivatePool.sol:421](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L421)
- [PrivatePool.sol:489](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489)
- [PrivatePool.sol:500](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L500)
- [PrivatePool.sol:522](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L522)
- [PrivatePool.sol:635](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635)
- [PrivatePool.sol:651](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651)
- [PrivatePool.sol:733](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L733)
- [PrivatePool.sol:744](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L744)
- [Factory.sol:87](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87)
- [Factory.sol:110](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L110)
- [Factory.sol:149](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L149)
- [PrivatePoolMetadata.sol:47](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47)
- [PrivatePoolMetadata.sol:103](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103)

### Example

```js
function assemblyOwnerNotZero(address _addr) public pure {
    assembly {
        if iszero(_addr) {
            mstore(0x00, "zero address")
            revert(0x00, 0x20)
        }
    }
}
```

## [G-03] Right shift or Left shift instead of dividing or multiplying by powers of two

### Lines

- [PrivatePool.sol:668](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L668)
- [PrivatePool.sol:736](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L736)

## [G-04] Use `calldata` instead of `memory` for function arguments that do not get mutated.

Mark data types as `calldata` instead of `memory` where possible. This makes it so that the data is not automatically loaded into memory. If the data passed into the function does not need to be changed (like updating values in an array), it can be passed in as `calldata`. The one exception to this is if the argument must later be passed into another function that takes an argument that specifies `memory` storage.

### Lines

- [PrivatePool.sol:386](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L386)
- [PrivatePool.sol:387](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L387)
- [PrivatePool.sol:388](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L388)
- [PrivatePool.sol:389](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L389)
- [PrivatePool.sol:390](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L390)
- [PrivatePool.sol:391](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L391)
- [PrivatePool.sol:392](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L392)
- [PrivatePool.sol:459](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459)
- [PrivatePool.sol:662](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L662)
- [PrivatePool.sol:663](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L663)
- [PrivatePool.sol:664](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L664)
- [PrivatePoolMetadata.sol:112](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112)

## [G-05] Use assembly to write storage values

### Lines

- [PrivatePool.sol:175-183](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L175-L183)
- [PrivatePool.sol:186](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L186)
- [PrivatePool.sol:540](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L540)
- [PrivatePool.sol:541](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L541)
- [PrivatePool.sol:552](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L552)
- [PrivatePool.sol:567](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L567)
- [PrivatePool.sol:578](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L578)
- [PrivatePool.sol:589](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L589)
- [Factory.sol:130](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L130)
- [Factory.sol:136](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L136)
- [Factory.sol:142](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L142)

### Example

```js
contract ExampleContract {
    address owner = 0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84;

    function assemblyUpdateOwner(address newOwner) public {
        assembly {
            sstore(owner.slot, newOwner)
        }
    }
}
```

## [G-06] Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas. Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

### Lines

- [PrivatePoolMetadata.sol:47](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47)
- [PrivatePoolMetadata.sol:103](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103)
- [EthRouter.sol:141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L141)
- [EthRouter.sol:142](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L142)
- [EthRouter.sol:203](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L203)
- [EthRouter.sol:208](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L208)
- [EthRouter.sol:290](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L290)
- [EthRouter.sol:291](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L291)

### Example

```js
contract ExampleContract1 {
    function assemblyInternalBalance() public returns (uint256) {
        assembly {
            let c := selfbalance()
            mstore(0x00, c)
            return(0x00, 0x20)
        }
    }
}

contract ExampleContract2 {
    function assemblyExternalBalance(address addr) public {
        uint256 bal;
        assembly {
            bal := balance(addr)
        }
    }
}
```

## [G-07] Pack structs

When creating structs, make sure that the variables are listed in ascending order by data type. The compiler will pack the variables that can fit into one 32 byte slot. If the variables are not listed in ascending order, the compiler may not pack the data into one slot, causing additional `sload` and `sstore` instructions when reading/storing the struct into the contract's storage.

### Lines

- [EthRouter.sol:48](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L48-L56)
- [EthRouter.sol:58](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L58-L67)

## [G-08] Use assembly to hash instead of Solidity

### Lines

- [PrivatePool.sol:642](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L642)
- [PrivatePool.sol:675](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L675)

### Example

```js
contract ExampleContract {
    function assemblyHash(uint256 a, uint256 b) public view {
        //optimized
        assembly {
            mstore(0x00, a)
            mstore(0x20, b)
            let hashedVal := keccak256(0x00, 0x40)
        }
    }
}
```

## [G-09] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety.

### Lines

- [PrivatePool.sol:230](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230)
- [PrivatePool.sol:236](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L236)
- [PrivatePool.sol:268](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L268)
- [PrivatePool.sol:323](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323)
- [PrivatePool.sol:335](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335)
- [PrivatePool.sol:429](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L429)
- [PrivatePool.sol:435](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L435)
- [PrivatePool.sol:436](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L436)
- [PrivatePool.sol:701](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L701)
- [PrivatePool.sol:703](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L703)
- [PrivatePool.sol:704](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L704)
- [PrivatePool.sol:705](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L705)
- [PrivatePool.sol:719](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L719)
- [PrivatePool.sol:721](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L721)
- [PrivatePool.sol:722](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L722)
- [PrivatePool.sol:723](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L723)
- [PrivatePool.sol:733](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L733)
- [PrivatePool.sol:734](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L734)
- [PrivatePool.sol:736](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L736)
- [PrivatePool.sol:737](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L737)
- [PrivatePool.sol:744](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L744)
- [PrivatePool.sol:745](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L745)
- [EthRouter.sol:115](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L115)
- [EthRouter.sol:182](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L182)

### Example

```js
contract ExampleAdd {
    //addition in assembly
    function addAssembly(uint256 a, uint256 b) public pure {
        assembly {
            let c := add(a, b)

            if lt(c, a) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }
        }
    }
}

contract ExampleSub {
    //subtraction in assembly
    function subAssembly(uint256 a, uint256 b) public pure {
        assembly {
            let c := sub(a, b)

            if gt(c, a) {
                mstore(0x00, "underflow")
                revert(0x00, 0x20)
            }
        }
    }
}

contract ExampleMul {
    //multiplication in assembly
    function mulAssembly(uint256 a, uint256 b) public pure {
        assembly {
            let c := mul(a, b)

            if lt(c, a) {
                mstore(0x00, "overflow")
                revert(0x00, 0x20)
            }
        }
    }
}

contract ExampleDiv {
    //division in assembly
    function divAssembly(uint256 a, uint256 b) public pure {
        assembly {
            let c := div(a, b)

            if gt(c, a) {
                mstore(0x00, "underflow")
                revert(0x00, 0x20)
            }
        }
    }
}
```
