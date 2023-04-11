## Summary
### Issues List
| Number |Issues Details|Context|
|:--:|:-------|:--:|
|[L-01]|Calculation of flash loan fee should be rounded up | 1 |
|[L-02]|Even if there is a fee of `msg.value`, the transaction will be reverted | 1 |
|[L-03]|Missing ReEntrancy Guard to ` safeTransferFrom `in functions | 6 |
|[L-04]|Prevent division by `0`|2|
|[L-05]|Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked| 1 |
|[L-06]|Use ```safeTransferOwnership``` instead of ```transferOwnership``` function | 1 |
|[L-07]|Use Fuzzing Test for math code bases | All Contracts |
|[L-08]|Project Upgrade and Stop Scenario should be | All Contracts |
|[L-09]|Insufficient coverage| All Contracts |
|[L-10]|Add a timelock to critical functions| 1 |
|[L-11] |Loss of precision due to rounding|1|
|[L-12] |Should an airdrop token arrive on the `private.sol` contract, it will be stuck| 1|
|[L-13] |Add to _blacklist function_| 1|
|[NC-14] |Missing Event for  initialize|3|
|[NC-15] |Constants on the left are better| 8 |
|[NC-16] |`Function writing` that does not comply with the `Solidity Style Guide`| All Contracts |
|[NC-17] |Include return parameters in NatSpec comments| All Contracts |
|[NC-18] |Tokens accidentally sent to the contract cannot be recovered | 1 |
|[NC-19] |` sell ` event is missing parameters| 1 |
|[NC-20] |Assembly Codes Specific – Should Have Comments | 1|
|[NC-21] |For functions, follow Solidity standard naming conventions (internal function style rule)| 1 |
|[NC-22] |Floating pragma| 5 |
|[NC-23] |Use SMTChecker| All Contract |
|[NC-24] |Lines are too long| 3 |
|[NC-25] |Omissions in Events| 8 |

Total 25 issues

### [L-01] Calculation of flash loan fee should be rounded up 

Math rounding should never favor the user, the calculation of flash load fee should be rounded up..



```solidity
    uint56 public changeFee;

src/PrivatePool.sol:
  632:         uint256 fee = flashFee(token, tokenId);

 function flashFee(address, uint256) public view returns (uint256) {
        return changeFee;
    }

```


```solidity

import "@openzeppelin/contracts/utils/math/Math.sol";

uint256 public changeFee;

function flashFee(address token, uint256 tokenId) public view returns (uint256) {
    uint256 fee = changeFee;
    uint256 amount = getFlashLoanAmount(token, tokenId);
    uint256 feeAmount = amount * fee / 1e18;
    return Math.ceil(feeAmount); // Round up the fee calculation
}
```


### [L-02] Even if there is a fee of `msg.value`, the transaction will be reverted.

Even if there is a fee of `msg.value`, the transaction will be reverted.

While using Flashloan, the fee amount should be min msg.value, but it is not like this in codes.


```solidity


src/PrivatePool.sol:
  623:     function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data)
  624:         external
  625:         payable
  626:         returns (bool)
  627:     {
  628:         // check that the NFT is available for a flash loan
  629:         if (!availableForFlashLoan(token, tokenId)) revert NotAvailableForFlashLoan();
  630: 
  631:         // calculate the fee
  632:         uint256 fee = flashFee(token, tokenId);
  633: 
  634:         // if base token is ETH then check that caller sent enough for the fee
- 635:         if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
+ 635:         if (baseToken == address(0) && msg.value <= fee) revert InvalidEthAmount();

  636: 
  637:         // transfer the NFT to the borrower
  638:         ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);
  639: 
  640:         // call the borrower
  641:         bool success =
  642:             receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");
  643: 
  644:         // check that flashloan was successful
  645:         if (!success) revert FlashLoanFailed();
  646: 
  647:         // transfer the NFT from the borrower
  648:         ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);
  649: 
  650:         // transfer the fee from the borrower
  651:         if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);
  652: 
  653:         return success;
  654:     }


```

### [L-03] Missing ReEntrancy Guard to ` safeTransferFrom `in functions

### Impact

EthRouter.sol  src/PrivatePool.sol and contracts have no Re-Entrancy protection in `safeTransferFrom` in functions


```solidity

6 results - 2 files

src/EthRouter.sol:
  239          for (uint256 i = 0; i < tokenIds.length; i++) {
  240:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
  241          }

src/PrivatePool.sol:
  239              // transfer the NFT to the caller
  240:             ERC721(nft).safeTransferFrom(address(this), msg.sender, tokenIds[i]);
  241  

  330              // transfer each nft from the caller
  331:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
  332  

  441          for (uint256 i = 0; i < inputTokenIds.length; i++) {
  442:             ERC721(nft).safeTransferFrom(msg.sender, address(this), inputTokenIds[i]);
  443          }

  446          for (uint256 i = 0; i < outputTokenIds.length; i++) {
  447:             ERC721(nft).safeTransferFrom(address(this), msg.sender, outputTokenIds[i]);
  448          }

  496          for (uint256 i = 0; i < tokenIds.length; i++) {
  497:             ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenIds[i]);
  498          }


```

if the mint was initiated by a contract, then the contract is checked for its ability to receive ERC721 tokens. Without reentrancy guard, onERC721Received will allow an attacker controlled contract to call the mint again, which may not be desirable to some parties, like allowing minting more than allowed.
https://www.paradigm.xyz/2021/08/the-dangers-of-surprising-code




### Proof of Concept
If `withdraw` is msg.sender contract, it can do re-entrancy by overriding `onERC721Received` function, it doesn't seem to be a serious problem since it conforms to check-effect-interaction pattern, but this is a clear re-entry due to access to other functions and pre-emit processing. is the entracy



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

### Recommended Mitigation Steps
Use Openzeppelin or Solmate Re-Entrancy pattern
Here is a example of a re-entrancy guard

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

### [L-04] Prevent division by `0`

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.
These functions can be calledd with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

` virtualNftReserves `  and ` (virtualNftReserves + inputAmount); ` values are should be checked for 0 value

```solidity

2 results - 1 file

src/PrivatePool.sol:
  719:         uint256 outputAmount = inputAmount * virtualBaseTokenReserves / (virtualNftReserves + inputAmount);
  745:         return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;


```


### [L-05] Function Calls in Loop Could Lead to Denial of Service due to Array length not being checked


```diff

src/EthRouter.sol:
   99:     function buy(Buy[] calldata buys, uint256 deadline, bool payRoyalties) public payable {
  100:         // check that the deadline has not passed (if any)
  101:         if (block.timestamp > deadline && deadline != 0) {
  102:             revert DeadlinePassed();
  103:         }
+     	 if buys.length > maxArrayLength) {
+          	      revert maxArrayLengt();
+       	 }

  104: 
  105:         // loop through and execute the the buys
  106:         for (uint256 i = 0; i < buys.length; i++) {
  107:             if (buys[i].isPublicPool) {
  108:                 // execute the buy against a public pool
  109:                 uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(
  110:                     buys[i].tokenIds, buys[i].baseTokenAmount, 0
  111:                 );
  112: 
  113:                 // pay the royalties if buyer has opted-in
  114:                 if (payRoyalties) {
  115:                     uint256 salePrice = inputAmount / buys[i].tokenIds.length;
  116:                     for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
  117:                         // get the royalty fee and recipient
  118:                         (uint256 royaltyFee, address royaltyRecipient) =
  119:                             getRoyalty(buys[i].nft, buys[i].tokenIds[j], salePrice);
  120: 
  121:                         if (royaltyFee > 0) {
  122:                             // transfer the royalty fee to the royalty recipient
  123:                             royaltyRecipient.safeTransferETH(royaltyFee);
  124:                         }
  125:                     }
  126:                 }
  127:             } else {
  128:                 // execute the buy against a private pool
  129:                 PrivatePool(buys[i].pool).buy{value: buys[i].baseTokenAmount}(
  130:                     buys[i].tokenIds, buys[i].tokenWeights, buys[i].proof
  131:                 );
  132:             }
  133: 
  134:             for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
  135:                 // transfer the NFT to the caller
  136:                 ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);
  137:             }
  138:         }
  139: 
  140:         // refund any surplus ETH to the caller
  141:         if (address(this).balance > 0) {
  142:             msg.sender.safeTransferETH(address(this).balance);
  143:         }
  144:     }

```

**Recommendation:**
Function calls made in unbounded loop are error-prone with potential resource exhaustion as it can trap the contract due to gas limitations or failed transactions. Consider bounding the loop length if the array is expected to be growing and/or handling a huge list of elements to avoid unnecessary gas wastage and denial of service.


### [L-06] Use ```safeTransferOwnership``` instead of ```transferOwnership``` function

**Context:**
```solidity
src/Factory.sol:
  26: import {Owned} from "solmate/auth/Owned.sol";

  37: contract Factory is ERC721, Owned {

```

**Description:**
```transferOwnership``` function is used to change Ownership from ``` solmate/blob/main/src/auth/Owned.sol ```.

Use a 2 structure transferOwnership which is safer. 
```safeTransferOwnership```,  use it is more secure due to 2-stage ownership transfer.

**Recommendation:**
Use ``Ownable2Step.sol``
[Ownable2Step.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol)

### [L-07]  Use Fuzzing Test for math code bases 


I recommend fuzzing testing in math code structures,

**Recommendation:**

Use should fuzzing test like Echidna.


Fuzzing is not easy, the tools are rough, and the math is hard, but it is worth it. Fuzzing gives me a level of confidence in my smart contracts that I didn’t have before. Relying just on unit testing anymore and poking around in a testnet seems reckless now.



https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05


### [L-08] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

### [L-09] Insufficient coverage

**Description:**
The test coverage rate of the project isn’t definded Testing all functions is best practice in terms of security criteria.
Due to its capacity, test coverage is expected to be 100%.


```js

README.md:
  113: - What is the overall line coverage percentage provided by your tests?:  N/a

```


Moreover , 
There is a bug related to generating coverage reports with forge. More info can be found here: foundry-rs/foundry#3357

### [L-10] Add a timelock to critical functions

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).
Consider adding a timelock to:

```solidity
src/Factory.sol:
  141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
  142:         protocolFeeRate = _protocolFeeRate;
  143:     }

```


### [L-11] Loss of precision due to rounding

Add scalars so roundings are negligible


```solidity
src/PrivatePool.sol:
  741      /// @return price The price of the pool.
  742:     function price() public view returns (uint256) {
  743:         // ensure that the exponent is always to 18 decimals of accuracy
  744:         uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());
  745:         return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;
  746:     }

```
### [L-12] Should an airdrop token arrive on the `private.sol` contract, it will be stuck

With the `buy()` `change()`   `sell()`functions, NFTs are transferred to the contract and in case of airdrop due to these NFTs, it will be stuck in the contract as there is no function to take these airdrop tokens from the contract.

Important NFT project owners are given airdrops, especially since the project includes NFTs such as BAYC, Moonbirds, Doodles, Azuki, there is a high probability of receiving Airdrops, but there is no function to withdraw incoming airdrop tokens, so airdrop tokens will be stuck in the contract.



### Recommended Mitigation Steps

Add this code:

```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```


###  [L-13] Add to _blacklist function_

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity.
To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.


Here is the project example; Manifold

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

```js
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }
```
Recommended Mitigation Steps add to Blacklist function and modifier.

### [N-14] Missing Event for  initialize

**Context:**
```solidity

src/PrivatePool.sol:
  143:     constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
  144:         factory = payable(_factory);
  145:         royaltyRegistry = _royaltyRegistry;
  146:         stolenNftOracle = _stolenNftOracle;
  147:     }

src/Factory.sol:
  53:     constructor() ERC721("Caviar Private Pools", "POOL") Owned(msg.sender) {}

src/EthRouter.sol:
  90:     constructor(address _royaltyRegistry) {
  91:         royaltyRegistry = _royaltyRegistry;
  92:     }

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit

### [N-15] Constants on the left are better

If you use the constant first you support structures that veil programming errors. And one should only produce code either to add functionality, fix an programming error or trying to support structures to avoid programming errors (like design patterns). 

https://www.moserware.com/2008/01/constants-on-left-are-better-but-this.html

```solidity
8 results - 3 files

src/Factory.sol:
   87:         if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
  110:         if (_baseToken == address(0)) {
  149:         if (token == address(0)) {

src/PrivatePool.sol:
  357:         if (baseToken == address(0)) {
  489:         if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
  522:         if (token == address(0)) {
  635:         if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

src/PrivatePoolMetadata.sol:
  47:             trait("Base token balance",  Strings.toString(privatePool.baseToken() == address(0) ? address(privatePool).balance : ERC20(privatePool.baseToken()).balanceOf(address(privatePool))))



```



### [N-16] `Function writing` that does not comply with the `Solidity Style Guide`

**Context:**
All Contracts

**Description:**
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier.
But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

 constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

### [N-17] Include return parameters in NatSpec comments

**Context:**
All Contracts

**Description:**
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the Solidity official documentation. In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability.

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

**Recommendation:**
Include return parameters in NatSpec comments

_Recommendation  Code Style: (from Uniswap3)_
```js
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

### [N-18] Tokens accidentally sent to the contract cannot be recovered

src/PrivatePool.sol

It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

### Recommended Mitigation Steps

Add this code:

```solidity
contracts/RubiconMarket.sol

 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
  }
}

```


### [N-19] ` sell ` event is missing parameters


The ` PrivatePool.sol` contract has very important function; ` sell()` 


However, only amounts are published in emits, whereas msg.sender must be specified in every transaction, a contract or web page listening to events cannot react to users, emit does not serve the purpose.
Basically, this event cannot be used


```solidity

src/PrivatePool.sol:
  301:     function sell(
  302:         uint256[] calldata tokenIds,
  303:         uint256[] calldata tokenWeights,
  304:         MerkleMultiProof calldata proof,
  305:         IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
  306:     ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
  307:         // ~~~ Codes ~~~ //
  308: 
  370: 
  371:         // emit the sell event
- 372:         emit Sell(tokenIds, tokenWeights, netOutputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);
+ 372:         emit Sell(msg.sender, tokenIds, tokenWeights, netOutputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);


```



### Recommended Mitigation Steps
add `msg.sender` parameter in event-emit


### [N-20] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does


This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

```solidity

src/PrivatePool.sol:
  467:         if (returnData.length > 0) {
  468:             // solhint-disable-next-line no-inline-assembly
  469:             assembly {
  470:                 let returnData_size := mload(returnData)
  471:                 revert(add(32, returnData), returnData_size)
  472:             }
  473:         } else {
  474:             revert();
  475:         }
  476:     }

```


### [N-21] For functions, follow Solidity standard naming conventions (internal function style rule)

**Context:**
```solidity

src/PrivatePoolMetadata.sol:
  110      }
  111: 
  112:     function trait(string memory traitType, string memory value) internal pure returns (string memory) {
  113:         // forgefmt: disable-next-item
  114:         return string(
  115:             abi.encodePacked(
  116:                 '{ "trait_type": "', traitType, '",', '"value": "', value, '" }'
  117:             )
  118:         );a
  119:     }


```


**Description:**
The above codes don't follow Solidity's standard naming convention,

internal and private functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)


### [N-22] Floating pragma

**Description:**
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
https://swcregistry.io/docs/SWC-103

Floating Pragma List: 
```solidity
5 files

src/EthRouter.sol:
   1  // SPDX-License-Identifier: MIT
   2: pragma solidity ^0.8.19;

src/Factory.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.19;

src/PrivatePool.sol:
   1  // SPDX-License-Identifier: MIT
   2: pragma solidity ^0.8.19;

src/PrivatePoolMetadata.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.19;

src/interfaces/IStolenNftOracle.sol:
  1  // SPDX-License-Identifier: MIT
  2: pragma solidity ^0.8.19;

```


**Recommendation:**
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

### [N-23] Use SMTChecker

The *highest* tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

### [N-24] Lines are too long

Usually lines in source code are limited to 80 characters. Today's screens are much larger so it's reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the lines below should be split when they reach that length

Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

There are many examples, some of which are listed below;

[PrivatePool.sol#L58-L60](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L58-L60)

[RubiconMarket.sol#L250](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47)

[PrivatePoolMetadata.sol#L103](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103)


### [N-25] Omissions in Events

Throughout the codebase, events are generally emitted when sensitive changes are made to the contracts. However, some events are missing important parameters

The events should include the new value and old value where possible:

```solidity

8 results - 2 files

src/Factory.sol:
  129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
  135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
  141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {

src/PrivatePool.sol:
  538:     function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner {
  550:     function setMerkleRoot(bytes32 newMerkleRoot) public onlyOwner {
  562:     function setFeeRate(uint16 newFeeRate) public onlyOwner {
  576:     function setUseStolenNftOracle(bool newUseStolenNftOracle) public onlyOwner {
  587:     function setPayRoyalties(bool newPayRoyalties) public onlyOwner {

```
