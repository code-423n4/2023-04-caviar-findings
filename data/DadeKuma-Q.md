
## Summary

---

### Low Findings
|Id|Title|Instances|
|:--:|:-------|:--:|
|[L-01]| Lack of two-step update for critical addresses | 2 |
|[L-02]| Loss of precision on division | 1 |
|[L-03]| Missing events in sensitive functions | 4 |
|[L-04]| Gas griefing/theft is possible on an unsafe external call | 1 |
|[L-05]| Unused/empty `receive()`/`fallback()` function | 3 |

Total: 11 instances over 5 issues.

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

### [L-01] Lack of two-step update for critical addresses

Add a two-step process for any critical address changes.

*There are 2 instances of this issue.*


```solidity
File: src/Factory.sol

129: 		    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
130: 		        privatePoolMetadata = _privatePoolMetadata;
131: 		    }

135: 		    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
136: 		        privatePoolImplementation = _privatePoolImplementation;
137: 		    }

```

### [L-02] Loss of precision on division

Solidity doesn't support fractions, so division by large numbers could result in the quotient being zero.

To avoid this, it's recommended to `require` a minimum numerator amount to ensure that it is always greater than the denominator.

*There is 1 instance of this issue.*


```solidity
File: src/PrivatePool.sol

745: 		        return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;

```

### [L-03] Missing events in sensitive functions

Events should be emitted when sensitive changes are made to the contracts, but some functions lack them.

*There are 4 instances of this issue.*


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

### [L-04] Gas griefing/theft is possible on an unsafe external call

A low-level call will copy any amount of bytes to local memory. When bytes are copied from returndata to memory, the memory expansion cost is paid.

This means that when using a standard solidity call, the callee can 'returnbomb' the caller, imposing an arbitrary gas cost.

Because this gas is paid by the caller and in the caller's context, it can cause the caller to run out of gas and halt execution.

Consider replacing all unsafe `call` with `excessivelySafeCall` from this [repository](https://github.com/nomad-xyz/ExcessivelySafeCall).

*There is 1 instance of this issue.*


```solidity
File: src/PrivatePool.sol

461: 		        (bool success, bytes memory returnData) = target.call{value: msg.value}(data);

```

### [L-05] Unused/empty `receive()`/`fallback()` function

If the intention is for the ETH to be used, the function should call another function, otherwise it should revert (e.g. `require(msg.sender == address(weth))`)

*There are 3 instances of this issue.*


```solidity
File: src/EthRouter.sol

88: 		    receive() external payable {}


File: src/Factory.sol

55: 		    receive() external payable {}


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


```solidity
File: src/PrivatePoolMetadata.sol

112: 		    function trait(string memory traitType, string memory value) internal pure returns (string memory) {

```

### [NC-04] Use of floating pragma

Locking the pragma helps avoid accidental deploys with an outdated compiler version that may introduce bugs and unexpected vulnerabilities.

Floating pragma is meant to be used for libraries and contracts that have external users and need backward compatibility.

*There are 5 instances of this issue.*


```solidity
File: src/EthRouter.sol

2: 		pragma solidity ^0.8.19;


File: src/Factory.sol

2: 		pragma solidity ^0.8.19;


File: src/PrivatePool.sol

2: 		pragma solidity ^0.8.19;


File: src/PrivatePoolMetadata.sol

2: 		pragma solidity ^0.8.19;


File: src/interfaces/IStolenNftOracle.sol

2: 		pragma solidity ^0.8.19;

```
