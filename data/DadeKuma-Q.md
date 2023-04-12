
## Summary

---

### Low Findings
|Id|Title|Instances|
|:--:|:-------|:--:|
|[L-01]| Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists | 24 |
|[L-02]| Lack of two-step update for critical addresses | 2 |
|[L-03]| Loss of precision on division | 1 |
|[L-04]| Missing events in sensitive functions | 4 |
|[L-05]| Gas griefing/theft is possible on an unsafe external call | 1 |
|[L-06]| Unused/empty `receive()`/`fallback()` function | 3 |

Total: 35 instances over 6 issues.

### Non Critical Findings
|Id|Title|Instances|
|:--:|:-------|:--:|
|[NC-01]| Use of `abi.encodePacked` instead of `bytes.concat` | 7 |
|[NC-02]| Parameter omission in events | 5 |
|[NC-03]| Some functions don't follow the Solidity naming conventions | 1 |
|[NC-04]| Use of floating pragma | 5 |

Total: 18 instances over 4 issues.

## Low Findings

---

### [L-01] Solmate's SafeTransferLib doesn't check whether the ERC20 contract exists

Solmate's SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).
This is stated in the [Solmate library](https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9).

*There are 24 instances of this issue.*

<details>
<summary>Expand findings</summary>

[src/EthRouter.sol#L123](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L123)

[src/EthRouter.sol#L142](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L142)

[src/EthRouter.sol#L190](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L190)

[src/EthRouter.sol#L208](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L208)

[src/EthRouter.sol#L291](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L291)


```solidity
File: src/EthRouter.sol

123: 		                            royaltyRecipient.safeTransferETH(royaltyFee);

142: 		            msg.sender.safeTransferETH(address(this).balance);

190: 		                            royaltyRecipient.safeTransferETH(royaltyFee);

208: 		        msg.sender.safeTransferETH(address(this).balance);

291: 		            msg.sender.safeTransferETH(address(this).balance);

```


[src/Factory.sol#L112](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L112)

[src/Factory.sol#L150](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L150)


```solidity
File: src/Factory.sol

112: 		            address(privatePool).safeTransferETH(baseTokenAmount);

150: 		            msg.sender.safeTransferETH(amount);

```


[src/PrivatePool.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L256)

[src/PrivatePool.sol#L259](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L259)

[src/PrivatePool.sol#L265](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L265)

[src/PrivatePool.sol#L268](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L268)

[src/PrivatePool.sol#L279](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L279)

[src/PrivatePool.sol#L281](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L281)

[src/PrivatePool.sol#L346](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L346)

[src/PrivatePool.sol#L348](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L348)

[src/PrivatePool.sol#L359](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L359)

[src/PrivatePool.sol#L362](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L362)

[src/PrivatePool.sol#L368](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L368)

[src/PrivatePool.sol#L423](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L423)

[src/PrivatePool.sol#L426](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L426)

[src/PrivatePool.sol#L432](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L432)

[src/PrivatePool.sol#L436](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L436)

[src/PrivatePool.sol#L502](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L502)

[src/PrivatePool.sol#L524](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L524)


```solidity
File: src/PrivatePool.sol

256: 		            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);

259: 		            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

265: 		            if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

268: 		            if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);

279: 		                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);

281: 		                        recipient.safeTransferETH(royaltyFee);

346: 		                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);

348: 		                        recipient.safeTransferETH(royaltyFee);

359: 		            msg.sender.safeTransferETH(netOutputAmount);

362: 		            if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

368: 		            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);

423: 		            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);

426: 		            if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);

432: 		            if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);

436: 		                msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);

502: 		            ERC20(baseToken).safeTransferFrom(msg.sender, address(this), baseTokenAmount);

524: 		            msg.sender.safeTransferETH(tokenAmount);

```
</details>

---
### [L-02] Lack of two-step update for critical addresses

Add a two-step process for any critical address changes.

*There are 2 instances of this issue.*

[src/Factory.sol#L129-L131](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L131)

[src/Factory.sol#L135-L137](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135-L137)


```solidity
File: src/Factory.sol

129: 		    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
130: 		        privatePoolMetadata = _privatePoolMetadata;
131: 		    }

135: 		    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
136: 		        privatePoolImplementation = _privatePoolImplementation;
137: 		    }

```

### [L-03] Loss of precision on division

Solidity doesn't support fractions, so division by large numbers could result in the quotient being zero.

To avoid this, it's recommended to `require` a minimum numerator amount to ensure that it is always greater than the denominator.

*There is 1 instance of this issue.*

[src/PrivatePool.sol#L745](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L745)


```solidity
File: src/PrivatePool.sol

745: 		        return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;

```

### [L-04] Missing events in sensitive functions

Events should be emitted when sensitive changes are made to the contracts, but some functions lack them.

*There are 4 instances of this issue.*

[src/Factory.sol#L129-L131](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129-L131)

[src/Factory.sol#L135-L137](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135-L137)

[src/Factory.sol#L141-L143](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141-L143)


```solidity
File: src/Factory.sol

129: 		    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
130: 		        privatePoolMetadata = _privatePoolMetadata;
131: 		    }

135: 		    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
136: 		        privatePoolImplementation = _privatePoolImplementation;
137: 		    }

141: 		    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
142: 		        protocolFeeRate = _protocolFeeRate;
143: 		    }

```


[src/PrivatePool.sol#L602-L615](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L602-L615)


```solidity
File: src/PrivatePool.sol

602: 		    function setAllParameters(
603: 		        uint128 newVirtualBaseTokenReserves,
604: 		        uint128 newVirtualNftReserves,
605: 		        bytes32 newMerkleRoot,
606: 		        uint16 newFeeRate,
607: 		        bool newUseStolenNftOracle,
608: 		        bool newPayRoyalties
609: 		    ) public {
610: 		        setVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);
611: 		        setMerkleRoot(newMerkleRoot);
612: 		        setFeeRate(newFeeRate);
613: 		        setUseStolenNftOracle(newUseStolenNftOracle);
614: 		        setPayRoyalties(newPayRoyalties);
615: 		    }

```

### [L-05] Gas griefing/theft is possible on an unsafe external call

A low-level call will copy any amount of bytes to local memory. When bytes are copied from returndata to memory, the memory expansion cost is paid.

This means that when using a standard solidity call, the callee can 'returnbomb' the caller, imposing an arbitrary gas cost.

Because this gas is paid by the caller and in the caller's context, it can cause the caller to run out of gas and halt execution.

Consider replacing all unsafe `call` with `excessivelySafeCall` from this [repository](https://github.com/nomad-xyz/ExcessivelySafeCall).

*There is 1 instance of this issue.*

[src/PrivatePool.sol#L461](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L461)


```solidity
File: src/PrivatePool.sol

461: 		        (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

```

### [L-06] Unused/empty `receive()`/`fallback()` function

If the intention is for the ETH to be used, the function should call another function, otherwise it should revert (e.g. `require(msg.sender == address(weth))`)

*There are 3 instances of this issue.*

[src/EthRouter.sol#L88](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L88)


```solidity
File: src/EthRouter.sol

88: 		    receive() external payable {}

```


[src/Factory.sol#L55](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L55)


```solidity
File: src/Factory.sol

55: 		    receive() external payable {}

```


[src/PrivatePool.sol#L134](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L134)


```solidity
File: src/PrivatePool.sol

134: 		    receive() external payable {}

```

## Non Critical Findings

---

### [NC-01] Use of `abi.encodePacked` instead of `bytes.concat`

Starting from version `0.8.4`, the recommended approach for appending bytes is to use `bytes.concat()` instead of `abi.encodePacked()`.

*There are 7 instances of this issue.*

<details>
<summary>Expand findings</summary>

[src/PrivatePoolMetadata.sol#L19-L28](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L19-L28)

[src/PrivatePoolMetadata.sol#L30](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L30)

[src/PrivatePoolMetadata.sol#L39-L48](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L39-L48)

[src/PrivatePoolMetadata.sol#L62-L76](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L62-L76)

[src/PrivatePoolMetadata.sol#L81-L92](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L81-L92)

[src/PrivatePoolMetadata.sol#L97-L106](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L97-L106)

[src/PrivatePoolMetadata.sol#L115-L117](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L115-L117)


```solidity
File: src/PrivatePoolMetadata.sol

19: 		        bytes memory metadata = abi.encodePacked(
20: 		            "{",
21: 		                '"name": "Private Pool ',Strings.toString(tokenId),'",',
22: 		                '"description": "Caviar private pool AMM position.",',
23: 		                '"image": ','"data:image/svg+xml;base64,', Base64.encode(svg(tokenId)),'",',
24: 		                '"attributes": [',
25: 		                    attributes(tokenId),
26: 		                "]",
27: 		            "}"
28: 		        );

30: 		        return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));

39: 		        bytes memory _attributes = abi.encodePacked(
40: 		            trait("Pool address", Strings.toHexString(address(privatePool))), ',',
41: 		            trait("Base token", Strings.toHexString(privatePool.baseToken())), ',',
42: 		            trait("NFT", Strings.toHexString(privatePool.nft())), ',',
43: 		            trait("Virtual base token reserves",Strings.toString(privatePool.virtualBaseTokenReserves())), ',',
44: 		            trait("Virtual NFT reserves", Strings.toString(privatePool.virtualNftReserves())), ',',
45: 		            trait("Fee rate (bps): ", Strings.toString(privatePool.feeRate())), ',',
46: 		            trait("NFT balance", Strings.toString(ERC721(privatePool.nft()).balanceOf(address(privatePool)))), ',',
47: 		            trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))
48: 		        );

62: 		            _svg = abi.encodePacked(
63: 		                '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 400 400" style="width:100%;background:black;fill:white;font-family:serif;">',
64: 		                    '<text x="24px" y="24px" font-size="12">',
65: 		                        "Caviar AMM private pool position",
66: 		                    "</text>",
67: 		                    '<text x="24px" y="48px" font-size="12">',
68: 		                        "Private pool: ", Strings.toHexString(address(privatePool)),
69: 		                    "</text>",
70: 		                    '<text x="24px" y="72px" font-size="12">',
71: 		                        "Base token: ", Strings.toHexString(privatePool.baseToken()),
72: 		                    "</text>",
73: 		                    '<text x="24px" y="96px" font-size="12">',
74: 		                        "NFT: ", Strings.toHexString(privatePool.nft()),
75: 		                    "</text>"
76: 		            );

81: 		            _svg = abi.encodePacked(
82: 		                _svg,
83: 		                '<text x="24px" y="120px" font-size="12">',
84: 		                    "Virtual base token reserves: ", Strings.toString(privatePool.virtualBaseTokenReserves()),
85: 		                "</text>",
86: 		                '<text x="24px" y="144px" font-size="12">',
87: 		                    "Virtual NFT reserves: ", Strings.toString(privatePool.virtualNftReserves()),
88: 		                "</text>",
89: 		                '<text x="24px" y="168px" font-size="12">',
90: 		                    "Fee rate (bps): ", Strings.toString(privatePool.feeRate()),
91: 		                "</text>"
92: 		            );

97: 		            _svg = abi.encodePacked(
98: 		                _svg, 
99: 		                    '<text x="24px" y="192px" font-size="12">',
100: 		                        "NFT balance: ", Strings.toString(ERC721(privatePool.nft()).balanceOf(address(privatePool))),
101: 		                    "</text>",
102: 		                    '<text x="24px" y="216px" font-size="12">',
103: 		                        "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),
104: 		                    "</text>",
105: 		                "</svg>"
106: 		            );

115: 		            abi.encodePacked(
116: 		                '{ "trait_type": "', traitType, '",', '"value": "', value, '" }'
117: 		            )

```
</details>

---
### [NC-02] Parameter omission in events

Events are generally emitted when sensitive changes are made to the contracts.

However, some are missing important parameters, as they should include both the new value and old value where possible.

*There are 5 instances of this issue.*

[src/PrivatePool.sol#L544](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L544)

[src/PrivatePool.sol#L555](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L555)

[src/PrivatePool.sol#L570](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L570)

[src/PrivatePool.sol#L581](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L581)

[src/PrivatePool.sol#L592](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L592)


```solidity
File: src/PrivatePool.sol

544: 		        emit SetVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);

555: 		        emit SetMerkleRoot(newMerkleRoot);

570: 		        emit SetFeeRate(newFeeRate);

581: 		        emit SetUseStolenNftOracle(newUseStolenNftOracle);

592: 		        emit SetPayRoyalties(newPayRoyalties);

```

### [NC-03] Some functions don't follow the Solidity naming conventions

Follow the Solidity naming convention by adding an underscore before the function name, for any `private` and `internal` functions.

*There is 1 instance of this issue.*

[src/PrivatePoolMetadata.sol#L112](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L112)


```solidity
File: src/PrivatePoolMetadata.sol

112: 		    function trait(string memory traitType, string memory value) internal pure returns (string memory) {

```

### [NC-04] Use of floating pragma

Locking the pragma helps avoid accidental deploys with an outdated compiler version that may introduce bugs and unexpected vulnerabilities.

Floating pragma is meant to be used for libraries and contracts that have external users and need backward compatibility.

*There are 5 instances of this issue.*

[src/EthRouter.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L2)


```solidity
File: src/EthRouter.sol

2: 		pragma solidity ^0.8.19;

```


[src/Factory.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L2)


```solidity
File: src/Factory.sol

2: 		pragma solidity ^0.8.19;

```


[src/PrivatePool.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L2)


```solidity
File: src/PrivatePool.sol

2: 		pragma solidity ^0.8.19;

```


[src/PrivatePoolMetadata.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L2)


```solidity
File: src/PrivatePoolMetadata.sol

2: 		pragma solidity ^0.8.19;

```


[src/interfaces/IStolenNftOracle.sol#L2](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/interfaces/IStolenNftOracle.sol#L2)


```solidity
File: src/interfaces/IStolenNftOracle.sol

2: 		pragma solidity ^0.8.19;

```
