## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Add to `blacklist` function | 1 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Do not allow fees to be set to `100%` | 2 |
| [LOW&#x2011;3](#LOW&#x2011;3) | `decimals()` not part of ERC20 standard | 2 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Event is missing parameters | 7 |
| [LOW&#x2011;5](#LOW&#x2011;5) | Missing Contract-existence Checks Before Low-level Calls | 1 |
| [LOW&#x2011;6](#LOW&#x2011;6) | Missing ReEntrancy Guard to `withdraw` function | 1 |
| [LOW&#x2011;7](#LOW&#x2011;7) | Missing Checks for Address(0x0)  | 1 |
| [LOW&#x2011;8](#LOW&#x2011;8) | Contracts are not using their OZ Upgradeable counterparts | 5 |
| [LOW&#x2011;9](#LOW&#x2011;9) | Missing length check for inputs | 4 |
| [LOW&#x2011;10](#LOW&#x2011;10) | Protect your NFT from copying in POW forks | 2 |
| [LOW&#x2011;11](#LOW&#x2011;11) | `tokenURI()` does not follow EIP-721 | 2 |
| [LOW&#x2011;12](#LOW&#x2011;12) | Unused `receive()` Function Will Lock Ether In Contract  | 3 |

Total: 31 contexts over 12 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Add a timelock to critical functions | 12 |
| [NC&#x2011;2](#NC&#x2011;2) | Avoid Floating Pragmas: The Version Should Be Locked | 5 |
| [NC&#x2011;3](#NC&#x2011;3) | Critical Changes Should Use Two-step Procedure | 12 |
| [NC&#x2011;4](#NC&#x2011;4) | Event Is Missing Indexed Fields | 5 |
| [NC&#x2011;5](#NC&#x2011;5) | Imports can be grouped together | 24 |
| [NC&#x2011;6](#NC&#x2011;6) | NatSpec return parameters should be included in contracts | 1 |
| [NC&#x2011;7](#NC&#x2011;7) | No need to initialize uints to zero | 2 |
| [NC&#x2011;8](#NC&#x2011;8) | Initial value check is missing in Set Functions | 10 |
| [NC&#x2011;9](#NC&#x2011;9) | Missing event for critical parameter change | 5 |
| [NC&#x2011;10](#NC&#x2011;10) | Implementation contract may not be initialized | 3 |
| [NC&#x2011;11](#NC&#x2011;11) | Public Functions Not Called By The Contract Should Be Declared External Instead | 8 |
| [NC&#x2011;12](#NC&#x2011;12) | `require()` / `revert()` Statements Should Have Descriptive Reason Strings | 1 |
| [NC&#x2011;13](#NC&#x2011;13) | Use `bytes.concat()` | 7 |
| [NC&#x2011;14](#NC&#x2011;14) | Use of Block.Timestamp | 16 |

Total: 111 contexts over 14 issues

## Low Risk Issues

### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Add to `blacklist` function

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```


#### <ins>Recommended Mitigation Steps</ins>
Add to Blacklist function and modifier.



### <a href="#Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Do not allow fees to be set to `100%`

It is recommended from a risk perspective to disallow setting 100% fees at all. A reasonable fee maximum should be checked for instead.

#### <ins>Proof Of Concept</ins>


```solidity

function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L141

```solidity

function setFeeRate(uint16 newFeeRate) public onlyOwner {
        
        if (newFeeRate > 5_000) revert FeeRateTooHigh();

        
        feeRate = newFeeRate;

        
        emit SetFeeRate(newFeeRate);
    }

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L562







### <a href="#Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> `decimals()` not part of ERC20 standard

`decimals()` is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

#### <ins>Proof Of Concept</ins>


```solidity
733: uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals()
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L733

```solidity
744: uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals()
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L744






### <a href="#Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Event is missing parameters

The following functions are missing critical parameters when emitting an event.
When dealing with source address which uses the value of `msg.sender`, the `msg.sender` value must be specified in every transaction, a contract or web page listening to events cannot react to users, emit does not serve the purpose. Basically, this event cannot be used.

#### <ins>Proof Of Concept</ins>


```solidity

function create(
    ... 
        emit Create(address(privatePool), tokenIds, baseTokenAmount);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L124

```solidity

function withdraw(address token, uint256 amount) public onlyOwner {
    ...
        emit Withdraw(token, amount);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L155

```solidity

function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
    ... 
        emit Buy(tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L288

```solidity

function sell(
    ...
        emit Sell(tokenIds, tokenWeights, netOutputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L372

```solidity

function change(
    ...
        emit Change(inputTokenIds, inputTokenWeights, outputTokenIds, outputTokenWeights, feeAmount, protocolFeeAmount);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L451

```solidity

function deposit(uint256[] calldata tokenIds, uint256 baseTokenAmount) public payable {
    ...  
        emit Deposit(tokenIds, baseTokenAmount);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L506

```solidity

function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
    ...
        emit Withdraw(_nft, tokenIds, token, tokenAmount);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L531




#### <ins>Recommended Mitigation Steps</ins>
Add `msg.sender` parameter in event-emit





### <a href="#Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Missing Contract-existence Checks Before Low-level Calls

Low-level calls return success if there is no code present at the specified address. 

#### <ins>Proof Of Concept</ins>


```solidity
461: (bool success, bytes memory returnData) = target.call{value: msg.value}(data);
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L461




#### <ins>Recommended Mitigation Steps</ins>

In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0`



### <a href="#Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Missing ReEntrancy Guard to `withdraw` function

The following contracts have no Re-Entrancy protection in their withdrawal function

If the mint was initiated by a contract, then the contract is checked for its ability to receive ERC721 tokens. Without reentrancy guard, `onERC721Received` will allow an attacker controlled contract to call the `mint` again, which may not be desirable to some parties, like allowing minting more than allowed. 

Reference: https://www.paradigm.xyz/2021/08/the-dangers-of-surprising-code

#### <ins>Proof Of Concept</ins>

If withdraw is msg.sender contract, it can do re-entrancy by overriding onERC721Received function, it doesn't seem to be a serious problem since it conforms to check-effect-interaction pattern, but this is a clear re-entry due to access to other functions and pre-emit processing. is the entracy

```solidity
reentrancy.sol:
 function onERC721Received(
    address,
    address,
    uint256,
    bytes memory
  ) public virtual override returns (bytes4) {
    //...do something
    }
    return this.onERC721Received.selector;
  }
```

Affected lines: 

```solidity
514: function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner {
        

        
        for (uint256 i = 0; i < tokenIds.length; i++) {
            ERC721(_nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
        }

        if (token == address(0)) {
            
            msg.sender.safeTransferETH(tokenAmount);
        } else {
            
            ERC20(token).transfer(msg.sender, tokenAmount);
        }

        
        emit Withdraw(_nft, tokenIds, token, tokenAmount);
    }

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L514




#### <ins>Recommended Mitigation Steps</ins>

Use Openzeppelin or Solmate Re-Entrancy pattern Here is a example of a re-entrancy guard

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract ReEntrancyGuard {
    bool internal locked;

    modifier noReentrant() {
        require(!locked, "No re-entrancy");
        locked = true;
        _;
        locked = false;
    }
}
```



### <a href="#Summary">[LOW&#x2011;7]</a><a name="LOW&#x2011;7"> Missing Checks for Address(0x0) 

Lack of zero-address validation on address parameters may lead to transaction reverts, waste gas, require resubmission of transactions and may even force contract redeployments in certain cases within the protocol.

#### <ins>Proof Of Concept</ins>


```solidity
459: function execute: address target
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L459



#### <ins>Recommended Mitigation Steps</ins>

Consider adding explicit zero-address validation on input parameters of address type.



### <a href="#Summary">[LOW&#x2011;8]</a><a name="LOW&#x2011;8"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

```solidity
33: import {IERC2981} from "openzeppelin/interfaces/IERC2981.sol";

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L33

```solidity
32: import {IERC2981} from "openzeppelin/interfaces/IERC2981.sol";
34: import {IERC3156FlashBorrower} from "openzeppelin/interfaces/IERC3156FlashLender.sol";

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L32-L34

```solidity
4: import {Strings} from "openzeppelin/utils/Strings.sol";
5: import {Base64} from "openzeppelin/utils/Base64.sol";

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L4-L5



#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.
See https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/tree/master/contracts for list of available upgradeable contracts



### <a href="#Summary">[LOW&#x2011;9]</a><a name="LOW&#x2011;9"> Missing length check for inputs

The following functions are missing length checks for the inputs. Functions that contain two or more input arrays should validate that the length of both inputs are equal.

#### <ins>Proof Of Concept</ins>


```solidity
    function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L211

```solidity
    function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L301

```solidity
    function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L385

```solidity
    function sumWeightsAndValidateProof(
        uint256[] memory tokenIds,
        uint256[] memory tokenWeights,
        MerkleMultiProof memory proof
    ) public view returns (uint256) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L661

#### <ins>Recommended Mitigation Steps</ins>

Recommend making sure division by 0 won’t occur by checking the variables beforehand and handling this edge case.



### <a href="#Summary">[LOW&#x2011;10]</a><a name="LOW&#x2011;10"> Protect your NFT from copying in POW forks
Ethereum has performed the long-awaited "merge" that will dramatically reduce the environmental impact of the network

There may be forked versions of Ethereum, which could cause confusion and lead to scams as duplicated NFT assets enter the market.

If the Ethereum Merge, which took place in September 2022, results in the Blockchain splitting into two Blockchains due to the 'THE DAO' attack in 2016, this could result in duplication of immutable tokens (NFTs).

In any case, duplicate NFTs will exist due to the ETH proof-of-work chain and other potential forks, and there’s likely to be some level of confusion around which assets are 'official' or 'authentic.'

Even so, there could be a frenzy for these copies, as NFT owners attempt to flip the proof-of-work versions of their valuable tokens.

As ETHPOW and any other forks spin off of the Ethereum mainnet, they will yield duplicate versions of Ethereum’s NFTs. An NFT is simply a blockchain token, and it can work as a deed of ownership to digital items like artwork and collectibles. A forked Ethereum chain will thus have duplicated deeds that point to the same tokenURI

About Merge Replay Attack: https://twitter.com/elerium115/status/1558471934924431363?s=20&t=RRheaYJwo-GmSnePwofgag

#### <ins>Proof Of Concept</ins>


```solidity
161: function tokenURI(uint256 id) public view override returns (string memory) {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L161

```solidity
17: function tokenURI(uint256 tokenId) public view returns (string memory) {

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L17



#### <ins>Recommended Mitigation Steps</ins>

Add the following check:
```solidity
if(block.chainid != 1) { 
    revert(); 
}
```





### <a href="#Summary">[LOW&#x2011;11]</a><a name="LOW&#x2011;11"> `tokenURI()` does not follow EIP-721

The <a href="https://eips.ethereum.org/EIPS/eip-721">EIP</a> states that `tokenURI()` "Throws if `_tokenId` is not a valid NFT", which the code below does not do. If the NFT has not yet been minted, `tokenURI()` should revert

#### <ins>Proof Of Concept</ins>


```solidity
161: function tokenURI(uint256 id) public view override returns (string memory) {
        return PrivatePoolMetadata(privatePoolMetadata).tokenURI(id);
    }

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L161

```solidity
17: function tokenURI(uint256 tokenId) public view returns (string memory) {
        
        bytes memory metadata = abi.encodePacked(
            "{",
                '"name": "Private Pool ',Strings.toString(tokenId),'",',
                '"description": "Caviar private pool AMM position.",',
                '"image": ','"data:image/svg+xml;base64,', Base64.encode(svg(tokenId)),'",',
                '"attributes": [',
                    attributes(tokenId),
                "]",
            "}"
        );

        return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)));
    }

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L17






### <a href="#Summary">[LOW&#x2011;12]</a><a name="LOW&#x2011;12"> Unused `receive()` Function Will Lock Ether In Contract 

If the intention is for the Ether to be used, the function should call another function, otherwise it should revert

#### <ins>Proof Of Concept</ins>


```solidity
88: receive() external payable {}
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L88

```solidity
55: receive() external payable {}
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L55

```solidity
134: receive() external payable {}
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L134



#### <ins>Recommended Mitigation Steps</ins>

The function should call another function, otherwise it should revert



## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to the following functions:

#### <ins>Proof Of Concept</ins>


```solidity
254: function change(Change[] calldata changes, uint256 deadline) public payable {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L254

```solidity
129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L129

```solidity
135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L135

```solidity
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L141

```solidity
385: function change(
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L385

```solidity
538: function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L538

```solidity
550: function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L550

```solidity
562: function setFeeRate(uint16 newFeeRate) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L562

```solidity
576: function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L576

```solidity
587: function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L587

```solidity
602: function setAllParameters(
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L602

```solidity
731: function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L731







### <a href="#Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Avoid Floating Pragmas: The Version Should Be Locked

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### <ins>Proof Of Concept</ins>

```solidity
Found usage of floating pragmas ^0.8.19 of Solidity in [EthRouter.sol]
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L2

```solidity
Found usage of floating pragmas ^0.8.19 of Solidity in [Factory.sol]
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L2

```solidity
Found usage of floating pragmas ^0.8.19 of Solidity in [PrivatePool.sol]
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L2

```solidity
Found usage of floating pragmas ^0.8.19 of Solidity in [PrivatePoolMetadata.sol]
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L2

```solidity
Found usage of floating pragmas ^0.8.19 of Solidity in [IStolenNftOracle.sol]
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/interfaces/IStolenNftOracle.sol#L2









### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Critical Changes Should Use Two-step Procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure

#### <ins>Proof Of Concept</ins>

```solidity
254: function change(Change[] calldata changes, uint256 deadline) public payable {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L254

```solidity
129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L129

```solidity
135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L135

```solidity
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L141

```solidity
385: function change(
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L385

```solidity
538: function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L538

```solidity
550: function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L550

```solidity
562: function setFeeRate(uint16 newFeeRate) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L562

```solidity
576: function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L576

```solidity
587: function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L587

```solidity
602: function setAllParameters(
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L602

```solidity
731: function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L731



#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.






### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Event Is Missing Indexed Fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). 

Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

#### <ins>Proof Of Concept</ins>


```solidity
event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L59

```solidity
event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L60

```solidity
event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L61

```solidity
event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L63

```solidity
event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L64






### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Imports can be grouped together

Consider importing OZ first, then all interfaces, then all utils.

#### <ins>Proof Of Concept</ins>


```solidity
31: import {ERC721, ERC721TokenReceiver} from "solmate/tokens/ERC721.sol";
32: import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
33: import {IERC2981} from "openzeppelin/interfaces/IERC2981.sol";
34: import {Pair, ReservoirOracle} from "caviar/Pair.sol";
35: import {IRoyaltyRegistry} from "royalty-registry-solidity/IRoyaltyRegistry.sol";
37: import {PrivatePool} from "./PrivatePool.sol";
38: import {IStolenNftOracle} from "./interfaces/IStolenNftOracle.sol";

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L31

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L32

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L33

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L34

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L35

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L37

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L38



```solidity
27: import {ERC20} from "solmate/tokens/ERC20.sol";
28: import {ERC721, ERC721TokenReceiver} from "solmate/tokens/ERC721.sol";
29: import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
30: import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
31: import {MerkleProofLib} from "solady/utils/MerkleProofLib.sol";
32: import {IERC2981} from "openzeppelin/interfaces/IERC2981.sol";
33: import {IRoyaltyRegistry} from "royalty-registry-solidity/IRoyaltyRegistry.sol";
34: import {IERC3156FlashBorrower} from "openzeppelin/interfaces/IERC3156FlashLender.sol";
36: import {IStolenNftOracle} from "./interfaces/IStolenNftOracle.sol";
37: import {Factory} from "./Factory.sol";

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L27

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L28

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L29

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L30

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L31

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L32

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L33

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L34

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L36

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L37









### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> NatSpec return parameters should be included in contracts

It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html
#### <ins>Proof Of Concept</ins>


See `PrivatePool.change` for example.


#### <ins>Recommended Mitigation Steps</ins>

Include return parameters in NatSpec comments

Recommendation Code Style: (from Uniswap3)

```solidity
    /// @notice Adds liquidity for the given recipient/tickLower/tickUpper position
    /// @dev The caller of this method receives a callback in the form of IUniswapV3MintCallback#uniswapV3MintCallback
    /// in which they must pay any token0 or token1 owed for the liquidity. The amount of token0/token1 due depends
    /// on tickLower, tickUpper, the amount of liquidity, and the current price.
    /// @param recipient The address for which the liquidity will be created
    /// @param tickLower The lower tick of the position in which to add liquidity
    /// @param tickUpper The upper tick of the position in which to add liquidity
    /// @param amount The amount of liquidity to mint
    /// @param data Any data that should be passed through to the callback
    /// @return amount0 The amount of token0 that was paid to mint the given amount of liquidity. Matches the value in the callback
    /// @return amount1 The amount of token1 that was paid to mint the given amount of liquidity. Matches the value in the callback
    function mint(
        address recipient,
        int24 tickLower,
        int24 tickUpper,
        uint128 amount,
        bytes calldata data
    ) external returns (uint256 amount0, uint256 amount1);
```



### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> No need to initialize uints to zero

There is no need to initialize `uint` variables to zero as their default value is `0`

#### <ins>Proof Of Concept</ins>

```solidity
237: uint256 royaltyFeeAmount = 0;

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L237

```solidity
328: uint256 royaltyFeeAmount = 0;

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L328





### <a href="#Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Initial value check is missing in Set Functions

A check regarding whether the current value and the new value are the same should be added

#### <ins>Proof Of Concept</ins>

```solidity
129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
        privatePoolMetadata = _privatePoolMetadata;
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L129

```solidity
135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L135

```solidity
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L141


```solidity
538: function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
        
        virtualBaseTokenReserves = newVirtualBaseTokenReserves;
        virtualNftReserves = newVirtualNftReserves;

        
        emit SetVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L538

```solidity
550: function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
        
        merkleRoot = newMerkleRoot;

        
        emit SetMerkleRoot(newMerkleRoot);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L550

```solidity
562: function setFeeRate(uint16 newFeeRate) public onlyOwner {
        
        if (newFeeRate > 5_000) revert FeeRateTooHigh();

        
        feeRate = newFeeRate;

        
        emit SetFeeRate(newFeeRate);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L562

```solidity
576: function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
        
        useStolenNftOracle = newUseStolenNftOracle;

        
        emit SetUseStolenNftOracle(newUseStolenNftOracle);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L576

```solidity
587: function setPayRoyalties(bool newPayRoyalties) public onlyOwner {
        
        payRoyalties = newPayRoyalties;

        
        emit SetPayRoyalties(newPayRoyalties);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L587

```solidity
602: function setAllParameters(
        uint128 newVirtualBaseTokenReserves,
        uint128 newVirtualNftReserves,
        bytes32 newMerkleRoot,
        uint16 newFeeRate,
        bool newUseStolenNftOracle,
        bool newPayRoyalties
    ) public {
        setVirtualReserves(newVirtualBaseTokenReserves, newVirtualNftReserves);
        setMerkleRoot(newMerkleRoot);
        setFeeRate(newFeeRate);
        setUseStolenNftOracle(newUseStolenNftOracle);
        setPayRoyalties(newPayRoyalties);
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L602

```solidity
731: function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
        
        uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;
        uint256 feePerNft = changeFee * 10 ** exponent;

        feeAmount = inputAmount * feePerNft / 1e18;
        protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000;
    }
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L731






### <a href="#Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Missing event for critical parameter change

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### <ins>Proof Of Concept</ins>


```solidity
129: function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L129

```solidity
135: function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L135

```solidity
141: function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L141

```solidity
602: function setAllParameters(
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L602

```solidity
731: function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L731





### <a href="#Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### <ins>Proof Of Concept</ins>


```solidity
90: constructor(address _royaltyRegistry)
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/EthRouter.sol#L90

```solidity
53: constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender)
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L53

```solidity
143: constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle)
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L143








### <a href="#Summary">[NC&#x2011;11]</a><a name="NC&#x2011;11"> Public Functions Not Called By The Contract Should Be Declared External Instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public.

#### <ins>Proof Of Concept</ins>


```solidity
function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L129

```solidity
function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L135

```solidity
function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L141

```solidity
function tokenURI(uint256 id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L161

```solidity
function predictPoolDeploymentAddress(bytes32 salt) public view returns (address predictedAddress) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/Factory.sol#L168

```solidity
function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L459

```solidity
function setAllParameters(
        uint128 newVirtualBaseTokenReserves,
        uint128 newVirtualNftReserves,
        bytes32 newMerkleRoot,
        uint16 newFeeRate,
        bool newUseStolenNftOracle,
        bool newPayRoyalties
    ) public {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L602

```solidity
function flashFeeToken() public view returns (address) {
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L755








### <a href="#Summary">[NC&#x2011;12]</a><a name="NC&#x2011;12"> `require()` / `revert()` Statements Should Have Descriptive Reason Strings

#### <ins>Proof Of Concept</ins>


```solidity
474: revert();
```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePool.sol#L474






### <a href="#Summary">[NC&#x2011;13]</a><a name="NC&#x2011;13"> Use `bytes.concat()`

Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

#### <ins>Proof Of Concept</ins>


```solidity
19: bytes memory metadata = abi.encodePacked(
            "{",
                '"name": "Private Pool ',Strings.toString(tokenId),'",',
                '"description": "Caviar private pool AMM position.",',
                '"image": ','"data:image/svg+xml
30: return string(abi.encodePacked("data:application/json;base64,", Base64.encode(metadata)))
39: bytes memory _attributes = abi.encodePacked(
            trait("Pool address", Strings.toHexString(address(privatePool))), ',',
            trait("Base token", Strings.toHexString(privatePool.baseToken())), ',',
            trait("NFT", Strings.toHexString(privatePool.nft())), ',',
            trait("Virtual base token reserves",Strings.toString(privatePool.virtualBaseTokenReserves())), ',',
            trait("Virtual NFT reserves", Strings.toString(privatePool.virtualNftReserves())), ',',
            trait("Fee rate (bps): ", Strings.toString(privatePool.feeRate())), ',',
            trait("NFT balance", Strings.toString(ERC721(privatePool.nft()).balanceOf(address(privatePool)))), ',',
            trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))
        )
62: _svg = abi.encodePacked(
                '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 400 400" style="width:100%;background:black;fill:white;font-family:serif;">',
                    '<text x="24px" y="24px" font-size="12">',
                        "Caviar AMM private pool position",
                    "</text>",
                    '<text x="24px" y="48px" font-size="12">',
                        "Private pool: ", Strings.toHexString(address(privatePool)),
                    "</text>",
                    '<text x="24px" y="72px" font-size="12">',
                        "Base token: ", Strings.toHexString(privatePool.baseToken()),
                    "</text>",
                    '<text x="24px" y="96px" font-size="12">',
                        "NFT: ", Strings.toHexString(privatePool.nft()),
                    "</text>"
            )
81: _svg = abi.encodePacked(
                _svg,
                '<text x="24px" y="120px" font-size="12">',
                    "Virtual base token reserves: ", Strings.toString(privatePool.virtualBaseTokenReserves()),
                "</text>",
                '<text x="24px" y="144px" font-size="12">',
                    "Virtual NFT reserves: ", Strings.toString(privatePool.virtualNftReserves()),
                "</text>",
                '<text x="24px" y="168px" font-size="12">',
                    "Fee rate (bps): ", Strings.toString(privatePool.feeRate()),
                "</text>"
            )
97: _svg = abi.encodePacked(
                _svg, 
                    '<text x="24px" y="192px" font-size="12">',
                        "NFT balance: ", Strings.toString(ERC721(privatePool.nft()).balanceOf(address(privatePool))),
                    "</text>",
                    '<text x="24px" y="216px" font-size="12">',
                        "Base token balance: ", Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))),
                    "</text>",
                "</svg>"
            )
115: abi.encodePacked(
                '{ "trait_type": "', traitType, '",', '"value": "', value, '" }'
            )
        )

```

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L19

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L30

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L39

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L62

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L81

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L97

https://github.com/code-423n4/2023-04-caviar/tree/main/src/PrivatePoolMetadata.sol#L115






#### <ins>Recommended Mitigation Steps</ins>

Use `bytes.concat()` and upgrade to at least Solidity version 0.8.4 if required. 

