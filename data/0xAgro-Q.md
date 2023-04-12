# QA Report
## Finding Summary

[**Low Severity**](#Low-Severity)
1. [**Unchecked Cast May Overflow**](#1-Unchecked-Cast-May-Overflow)
2. [**Early Use of Compiler Version**](#2-Early-Use-of-Compiler-Version)
3. [**Single Step Owner Transfer**](#3-Single-Step-Owner-Transfer)
4. [**Tokens Without Decimals Will Not Give Price**](#4-Tokens-Without-Decimals-Will-Not-Give-Price)
5. [**Withdraw Event Manipulation**](#5-Withdraw-Event-Manipulation)
6. [**Private Pool Creation Event Manipulation**](#6-Private-Pool-Creation-Event-Manipulation)

[**Non-Critical**](#Non-Critical)
1. [**Incomplete NatSpec**](#1-Incomplete-NatSpec)
2. [**Inconsistent Named Returns**](#2-Inconsistent-Named-Returns)
3. [**Spelling Mistakes**](#3-Spelling-Mistakes)
4. [**Events Not Indexed**](#4-Events-Not-Indexed)
5. [**bytes.concat() Can Be Used Over abi.encodePacked()**](#5-bytesconcat-can-be-used-over-abiencodepacked)
6. [**Inconsistent ASCII Art Starting Point**](#6-Inconsistent-ASCII-Art-Starting-Point)
7. [**Unbounded Compiler Version**](#7-Unbounded-Compiler-Version)
8. [**Value Can Be Used Again**](#8-Value-Can-Be-Used-Again)
9. [**Documentation Should Describe Functionality**](#9-Documentation-Should-Describe-Functionality)
10. [**Assembly Should Be Documented**](#10-Assembly-Should-Be-Documented)
11. [**Magic Number Used**](#11-Magic-Number-Used)*

[**Style Guide Violations**](#Style-Guide-Violations)
1. [**Maximum Line Length**](#1-Maximum-Line-Length)
2. [**Order of Functions**](#2-Order-of-Functions)
3. [**Order of Layout**](#3-Order-of-Layout)
4. [**Single Quote String**](#4-Single-Quote-String)

# Low Severity

## 1. Unchecked Cast May Overflow

As of Solidity 0.8 overflows are handled automatically; however, not for casting. For example `uint32(4294967300)` will result in `4` without reversion. Consider using OpenZepplin's SafeCast library. Even if it seems as though a value cannot overflow, it is best to be safe.

*/src/PrivatePool.sol*

```solidity
230:	virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
231:	virtualNftReserves -= uint128(weightSum);
323:	virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
324:	virtualNftReserves += uint128(weightSum);
```

## 2. Early Use of Compiler Version

All functions in scope operate under Solidity `^0.8.19`. Solidity `0.8.19` as of the start of the contest, is a little over [month old](https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/).

By using a compiler version early you are volunteering as guinea pigs for potential bugs when it is not necessary. As an example in [Solidity 0.8.13](https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/), [a compiler bug](https://github.com/ethereum/solidity-blog/blob/499ab8abc19391be7b7b34f88953a067029a5b45/_posts/2022-06-15-inline-assembly-memory-side-effects-bug.md) was found 81 days after the release announcement (see links).

Consider downgrading contracts of version `0.8.19` to a more battle-tested Solidity version.

## 3. Single Step Owner Transfer

The codebase uses solmate's `Owned` contract ([ex](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L26)) which uses a single step owner transfer seen [here](https://github.com/transmissions11/solmate/blob/main/src/auth/Owned.sol#L39-L42): 

```Solidity
39:    function transferOwnership(address newOwner) public virtual onlyOwner {
40:            owner = newOwner;
41:    
42:            emit OwnershipTransferred(msg.sender, newOwner);
43:    }
```

Single step owner transfers are prone to admin error.

Consider a 2 step ownership transfer like in [Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) to avoid possible transfer errors.

## 4. Tokens Without Decimals Will Not Give Price

Private Pools have a function [`price`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L742-L746) which assumes the `baseToken` has a `decimals` function. 

```Solidity
742:    function price() public view returns (uint256) {
743:        // ensure that the exponent is always to 18 decimals of accuracy
744:        uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
745:        return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;
746:    }
```

In regard to the `decimals` function, [EIP20](https://eips.ethereum.org/EIPS/eip-20) states: 
> "OPTIONAL - This method can be used to improve usability, but interfaces and other contracts MUST NOT expect these values to be present".

The [`price`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L742-L746) function will always revert if the `baseToken` does not implement a `decimals` function.

## 5. Withdraw Event Manipulation

In the Factory, the team can fake a withdraw event if they know how to trigger certain ERC20 Tokens. There is no check if the ERC20 `transfer` on [line 152](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L152) was successful (returns `true`).

```Solidity
148:    function withdraw(address token, uint256 amount) public onlyOwner {
149:        if (token == address(0)) {
150:            msg.sender.safeTransferETH(amount);
151:        } else {
152:            ERC20(token).transfer(msg.sender, amount);
153:        }
154:
155:        emit Withdraw(token, amount);
156:    }
```

For example, some Tokens like [Stasis Euro](https://etherscan.io/token/0xdb25f211ab05b1c97d595516f45794528a807ad8#code) return `false` (without reversion) if the user does not have the funds to transfer:

```Solidity
196:        uint256 fromBalance = accounts [msg.sender];
197:        if (fromBalance < _value) return false;
```

The team could call `withdraw`, no funds would be sent; however, a `Withdraw` event would be triggered indicating the full withdraw amount.

## 6. Private Pool Creation Event Manipulation

In the Factory, a user can fake the `baseTokenAmount` value in a `Create` event if they know how to trigger certain ERC20 Tokens. There is no check if the ERC20 `transferFrom` on [line 115](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L115) was successful (returns `true`).

```Solidity
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
84:    ) public payable returns (PrivatePool privatePool) {
85:        // check that the msg.value is equal to the base token amount if the base token is ETH or the msg.value is equal
86:        // to zero if the base token is not ETH
87:        if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
88:            revert PrivatePool.InvalidEthAmount();
89:        }
90:
91:        // deploy a minimal proxy clone of the private pool implementation
92:        privatePool = PrivatePool(payable(privatePoolImplementation.cloneDeterministic(_salt)));
93:
94:        // mint the nft to the caller
95:        _safeMint(msg.sender, uint256(uint160(address(privatePool))));
96:
97:        // initialize the pool
98:        privatePool.initialize(
99:            _baseToken,
100:           _nft,
101:           _virtualBaseTokenReserves,
102:           _virtualNftReserves,
103:           _changeFee,
104:           _feeRate,
105:           _merkleRoot,
106:           _useStolenNftOracle,
107:           _payRoyalties
108:       );
109:
110:       if (_baseToken == address(0)) {
111:           // transfer eth into the pool if base token is ETH
112:           address(privatePool).safeTransferETH(baseTokenAmount);
113:       } else {
114:           // deposit the base tokens from the caller into the pool
115:           ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);
116:       }
117:
118:       // deposit the nfts from the caller into the pool
119:       for (uint256 i = 0; i < tokenIds.length; i++) {
120:           ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
121:       }
122:
123:       // emit create event
124:       emit Create(address(privatePool), tokenIds, baseTokenAmount);
125:    }
```

For example, some Tokens like [Stasis Euro](https://etherscan.io/token/0xdb25f211ab05b1c97d595516f45794528a807ad8#code) return `false` (without reversion) if the user does not have the funds to transfer:

```Solidity
217:    uint256 spenderAllowance = allowances [_from][msg.sender];
218:    if (spenderAllowance < _value) return false;
```

The user could call `create`, no funds would be sent; however, a `Create` event would be triggered indicating the full `baseTokenAmount amount`.

# Non-Critical

## 1. Incomplete NatSpec

1 out of 5 of the contracts in scope are missing a `@title` tag. Given that 4 contracts all have a `@title` tag, consider adding one per the 1 remaining contracts.

[IStolenNftOracle.sol](https://github.com/code-423n4/2023-04-caviar/tree/main/src/interfaces/IStolenNftOracle.sol) is missing a `@title` tag.

The [Solidity Documentation](https://docs.soliditylang.org/en/v0.8.19/natspec-format.html#natspec-format) states: 

> "It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI)".

## 2. Inconsistent Named Returns

Some functions use named returns and others do not. It is best for code clearity to keep a consistent style.

1. The following contracts only have named returns (EX. `returns(uint256 foo)`): [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol).
2. The following contracts only have non-named returns (EX. `returns(uint256)`): [PrivatePoolMetadata.sol](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol).
3. The following contracts have both: [Factory.sol](https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol), and [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol).

## 3. Spelling Mistakes

There are some spelling mistakes throughout the codebase. Consider fixing all spelling mistakes.

*src/EthRouter.sol*

* The word `execute` is misspelled as [`exceute`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L169).

*src/PrivatePool.sol*

* The word `amount` is misspelled as [`aount`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L251).

## 4. Events Not Indexed

It is best practice to have 3 indexed fields in events. Indexed fields help off-chain tools analyze on-chain activity through filtering these indexed fields.

*/src/PrivatePool.sol*

```solidity
64:	event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);
65:	event SetMerkleRoot(bytes32 merkleRoot);
66:	event SetFeeRate(uint16 feeRate);
67:	event SetUseStolenNftOracle(bool useStolenNftOracle);
68:	event SetPayRoyalties(bool payRoyalties);
```

## 5. bytes.concat() Can Be Used Over abi.encodePacked()

Consider using `bytes.concat()` instead of `abi.encodePacked()` in contracts with Solidity version >= `0.8.4`.

*/src/PrivatePoolMetadata.sol*

```solidity
19:	bytes memory metadata = abi.encodePacked(
30:	return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));
39:	bytes memory _attributes = abi.encodePacked(
62:	_svg = abi.encodePacked(
81:	_svg = abi.encodePacked(
97:	_svg = abi.encodePacked(
115:	abi.encodePacked(
```

## 6. Inconsistent ASCII Art Starting Point

There are three instances of ASCII art in the codebase (in scope): a whale, and two other fishing graphics. The whale graphic starts after two empty comment lines; however, the two fishing graphics start after a single empty comment line. Consider removing the second empty comment line for the whale, seen [here](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L5).

*/src/Factory.sol*

```Solidity
4:    /*
5:     *
6:     *       __________...----..____..-'``-..___
7:     *     ,'.                                  ```--.._
8:     *    :                                             ``._
9:     *    |                           --                    ``.
10:    *    |                 -0-           -.     -   -.        `.
```

*/src/EthRouter.sol*

```Solidity
4:    /*
5:     *                                     _H_
6:     *                                    /___\
7:     *                                    \888/
8:     * ~^~^~^~^~^~^~^~^~^~^~^~^~^~^~^~^~^~^~U~^~^~^~^~^~^~^
9:     *                       ~              |
10:    *       ~                        o     |        ~
```

*/src/PrivatePool.sol*

```Solidity
4:    /*
5:     *                                   ____
6:     *                                /\|    ~~\
7:     *                              /'  |   ,-. `\
8:     *                             |       | X |  |
9:     *                            _|________`-'   |X
10:    *                          /'          ~~~~~~~~~,
```

## 7. Unbounded Compiler Version

All contracts in scope use a compiler version of `^0.8.19`. No one knows what will come in future compiler versions greater than `0.8.19` less than `0.9.0`, thus it is best to put an upper limit on the compiler version.

## 8. Value Can Be Used Again

In a Private Pool the value `netInputAmount - feeAmount - protocolFeeAmount` is calculated twice, [here](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230) and [here](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L236). Note the slight casting difference. Consider only calculating `netInputAmount - feeAmount - protocolFeeAmount` once.

## 9. Documentation Should Describe Functionality

In the Factory [`create`](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L71) function the `_baseToken` `@param` tag states:
> "The address of the base token". 

It should be noted in the `@param` tag that if the user would like to use ETH, they should set `_baseToken` to the zero address. The following code indicates this behavior: 

*/src/Factory.sol*

```Solidity
110:    if (_baseToken == address(0)) {
111:        // transfer eth into the pool if base token is ETH
112:        address(privatePool).safeTransferETH(baseTokenAmount);
113:    } else {
```

## 10. Assembly Should Be Documented

It is best practice to heavily comment all lines of assembly.

*/src/PrivatePool.sol*

```Solidity
469:    assembly {
470:        let returnData_size := mload(returnData)
471:        revert(add(32, returnData), returnData_size)
472:    }
```

## 11. Magic Number Used

*This issue adds missed items to the automated audit report found [here](https://gist.github.com/Picodes/f50f08a90e93acff6c069898839a7452#nc-1-constants-should-be-defined-rather-than-using-magic-numbers)*

It is best practice to replace all magic numbers with constants. `5_000` can be replaced with a constant called `MAX_FEE_RATE`.

*/src/PrivatePool.sol*

```Solidity
172:    if (_feeRate > 5_000) revert FeeRateTooHigh();
564:    if (newFeeRate > 5_000) revert FeeRateTooHigh();
```

# Style Guide Violations

## 1. Maximum Line Length

Lines with greater length than 120 characters are used. The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-lengthhttps://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length) suggests that all lines should be 120 characters or less in width.

The following lines are longer than 120 characters, it is suggested to shorten these lines:

*src/PrivatePoolMetadata.sol*
*  [47](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L47), [63](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L63), [103](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L103). 

*src/PrivatePool.sol*
*  [58](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L58), [59](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L59), [60](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L60), [63](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L63), [642](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L642). 

## 2. Order of Functions

The Solidity Style Guide suggests the following function order: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private.

The following contracts are not compliant (examples are only to prove the functions are out of order NOT a full description): 

* [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol): external functions are positioned after public functions.
* [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol): the Constructor is positioned after the receive function.

## 3. Order of Layout

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-layout) suggests the following contract layout order: Type declarations, State variables, Events, Modifiers, Functions.

The following contracts are not compliant (examples are only to prove the layout are out of order NOT a full description): 

* [Factory.sol](https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol): State is positioned after Events.
* [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol): State is positioned after Events.

## 4. Single Quote String

The [Solidity Style Guide](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#other-recommendations) states that: 
> "[s]trings should be quoted with double-quotes instead of single-quotes". 

Consider using double-quotes to be compliant if possible.

*/src/PrivatePoolMetadata.sol*

```solidity
21:	'"name": "Private Pool ',Strings.toString(tokenId),'",',
22:	'"description": "Caviar private pool AMM position.",',
23:	'"image": ','"data:image/svg+xml;base64,', Base64.encode(svg(tokenId)),'",',
24:	'"attributes": [',
40:	trait("Pool address", Strings.toHexString(address(privatePool))), ',',
41:	trait("Base token", Strings.toHexString(privatePool.baseToken())), ',',
42:	trait("NFT", Strings.toHexString(privatePool.nft())), ',',
43:	trait("Virtual base token reserves",Strings.toString(privatePool.virtualBaseTokenReserves())), ',',
44:	trait("Virtual NFT reserves", Strings.toString(privatePool.virtualNftReserves())), ',',
45:	trait("Fee rate (bps): ", Strings.toString(privatePool.feeRate())), ',',
46:	trait("NFT balance", Strings.toString(ERC721(privatePool.nft()).balanceOf(address(privatePool)))), ',',
64:	'<text x="24px" y="24px" font-size="12">',
67:	'<text x="24px" y="48px" font-size="12">',
70:	'<text x="24px" y="72px" font-size="12">',
73:	'<text x="24px" y="96px" font-size="12">',
83:	'<text x="24px" y="120px" font-size="12">',
86:	'<text x="24px" y="144px" font-size="12">',
89:	'<text x="24px" y="168px" font-size="12">',
99:	'<text x="24px" y="192px" font-size="12">',
102:	'<text x="24px" y="216px" font-size="12">',
116:	'{ "trait_type": "', traitType, '",', '"value": "', value, '" }'
```