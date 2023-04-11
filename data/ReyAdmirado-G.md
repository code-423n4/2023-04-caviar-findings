
| | issue |
| ----------- | ----------- |
| 1 | [Structs can be packed into fewer storage slots](#1-structs-can-be-packed-into-fewer-storage-slots) |
| 2 | [state variables should be cached in stack variables rather than re-reading them from storage](#2-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage) |
| 3 | [Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement](#3-add-unchecked--for-subtractions-where-the-operands-cannot-underflow-because-of-a-previous-require-or-if-statement) |
| 4 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#4-x--y-costs-more-gas-than-x--x--y-for-state-variables) |
| 5 | [can make the variable outside the loop to save gas](#5-can-make-the-variable-outside-the-loop-to-save-gas) |
| 6 | [usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#6-usage-of-uintint-smaller-than-32-bytes-256-bits-incurs-overhead) |
| 7 | [ Ternary over if ... else](#7-ternary-over-if--else) |
| 8 | [public functions not called by the contract should be declared external instead](#8-public-functions-not-called-by-the-contract-should-be-declared-external-instead) |
| 9 | [Use assembly to check for address(0)](#9-use-assembly-to-check-for-address0) |
| 10 | [Functions guaranteed to revert when called by normal users can be marked `payable`](#10-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable) |
| 11 | [Optimize names to save gas](#11-optimize-names-to-save-gas) |
| 12 | [Setting the constructor to payable](#12-setting-the-constructor-to-payable) |
| 13 | [part of the code can be pre calculated](#13-part-of-the-code-can-be-pre-calculated) |
| 14 | [Use selfbalance() instead of address(this).balance](#14-use-selfbalance-instead-of-addressthisbalance) |
| 15 | [Duplicated require()/revert() checks should be refactored to a modifier or function](#15-duplicated-requirerevert-checks-should-be-refactored-to-a-modifier-or-function) |
| 16 | [sorting the lines better will result in less storage access needed which result in gas save](#16-sorting-the-lines-better-will-result-in-less-storage-access-needed-which-result-in-gas-save) |
| 17 | [cache parts of code for second use](#17-cache-parts-of-code-for-second-use) |


## 1. Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

struct `Buy` can be while using one less slot if `isPublicPool` which is bool is made near `nft` which is address they can be packed into one slot instead of 2 slots
- [EthRouter.sol#L48-L56](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L48-L56)

same thing as before for struct Sell
- [EthRouter.sol#L58-L67](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L58-L67)


## 2. state variables should be cached in stack variables rather than re-reading them from storage

Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

cache `baseToken` before #L225 and if code reverts in that line we will just lose 3 gas for it but if it doesnt, worst case scenario we will save ~100 gas 2 of the 5 extra uses are inside a loop and gonna be read again per iteration so this will be huge save.
- [PrivatePool.sol#L225-L279](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225-L279)

cache `nft` and `payRoyalties` before #L238. 100 gas is being read for every iteration of for loop although `nft` and `payRoyalties` will not change, so cache them before the loop and used the cached versions inside.
- [PrivatePool.sol#L238-L242](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238-L242)

same thing for `nft` and `payRoyalties` here again but this time cache `nft` before #L316 because there is a possibilty for a 100 more gas save there
- [PrivatePool.sol#L316-333](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L316-333)

`baseToken` can be cached here for possibly a lot of gas save because its used in a loop and it is being read again later with some conditions, although it is very possible that its gonna save gas if its cached before #L328
- [PrivatePool.sol#L329-L369](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329-L369)

cache `baseToken` before #L397 for at least 97 gas save if the revert in #L397 doesnt happen and possibly ~300 gas save if conditions #L421 and #L426 work.
- [PrivatePool.sol#L397-L426](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397-L426)

cache `nft` because its being read every iteration in 2 for loops although its not being changed. cache it before #L400 though because there is a possibilty for ~100 gas save there too.
- [PrivatePool.sol#L441-L448](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441-L448)

`baseToken` is at least being read twice here or best case 4 times. caching it before #L489 at least saves 97 gas and possibly ~300.
- [PrivatePool.sol#L489-L502](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489-L502)

`nft` is being read every iteration of the loop here so cache it before the loop to save gas.
- [PrivatePool.sol#L496-L498](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L496-L498)

cache `baseToken` before #L635 for possible gas ~200 gas save if function doesnt revert in #L635.
- [PrivatePool.sol#L635-L651](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635-L651)

cache `baseToken` before #L733. this will risk losing 3 gas if `baseToken == address(0)` is true but otherwise it will save 97 gas.
- [PrivatePool.sol#L733](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L733)

same thing for #L744
- [PrivatePool.sol#L744](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L744)


## 3. Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
this will stop the check for overflow and underflow so it will save gas

`(netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;` can be done inside a unchecked bracket because its happened before so it cant underflow
- [PrivatePool.sol#L236](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L236)

`msg.value - netInputAmount` can be in unchecked because its being checked in the same line
- [PrivatePool.sol#L268](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L268)

`msg.value - feeAmount - protocolFeeAmount` can be unchecked because its checked before to not underflow
- [PrivatePool.sol#L436](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L436)

`18 - 4` can be unchecked
- [PrivatePool.sol#L733](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L733)

`36 - ERC20(baseToken).decimals()` can be unchecked because it cant underflow
- [PrivatePool.sol#L744](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L744)


## 4. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas (13 or 113 each dependant on the usage see [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8))

- [PrivatePool.sol#L231](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231)
- [PrivatePool.sol#L323](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323)
- [PrivatePool.sol#L230](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230)
- [PrivatePool.sol#L324](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324)


## 5. can make the variable outside the loop to save gas

make the variable outside the loop and only give the value to variable inside

- [EthRouter.sol#L118](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L118)
- [EthRouter.sol#L185](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L185)
- [EthRouter.sol#L262](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L262)

- [PrivatePool.sol#L274](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L274)


## 6. usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

many operations are being done on these kinds in - [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)


## 7. Ternary over if ... else

Using ternary operator instead of the if else statement saves gas.

- [Factory.sol#L110-L115](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L110-L115)
- [Factory.sol#L149-L154](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L149-L154)

- [PrivatePool.sol#L278-L282](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L278-L282)
- [PrivatePool.sol#L345-L349](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345-L349)
- [PrivatePool.sol#L522-L527](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L522-L527)


## 8. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

- [Factory.sol#L71](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L71)
- [Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129)
- [Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135)
- [Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141)
- [Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148)
- [Factory.sol#L168](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L168)

- [PrivatePoolMetadata.sol#L17](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17)

- [EthRouter.sol#L99](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L99)
- [EthRouter.sol#L219](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L219)
- [EthRouter.sol#L254](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254)


## 9. Use assembly to check for address(0)

saves 6 gas per instance

- [Factory.sol#L87](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87) 2 instances here
- [Factory.sol#L110](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L110)
- [Factory.sol#L149](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L149)

- [PrivatePoolMetadata.sol#L47](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47)
- [PrivatePoolMetadata.sol#L103](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103)

17 instances total in - [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

- [PrivatePool.sol#L225](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225)
- [PrivatePool.sol#L254](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L254)
- [PrivatePool.sol#L277](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277)
- [PrivatePool.sol#L278](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L278)
- [PrivatePool.sol#L344](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344)
- [PrivatePool.sol#L345](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345)
- [PrivatePool.sol#L357](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L357)
- [PrivatePool.sol#L397](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397)
- [PrivatePool.sol#L421](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L421)
- [PrivatePool.sol#L489](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489)
- [PrivatePool.sol#L500](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L500)
- [PrivatePool.sol#L522](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L522)
- [PrivatePool.sol#L635](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635)
- [PrivatePool.sol#L651](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651)
- [PrivatePool.sol#L733](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L733)
- [PrivatePool.sol#L744](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L744)


## 10. Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

- [Factory.sol#L129](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129)
- [Factory.sol#L135](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135)
- [Factory.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141)
- [Factory.sol#L148](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148)

- [PrivatePool.sol#L459](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459)
- [PrivatePool.sol#L514](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514)
- [PrivatePool.sol#L538](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538)
- [PrivatePool.sol#L550](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550)
- [PrivatePool.sol#L562](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562)
- [PrivatePool.sol#L576](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576)
- [PrivatePool.sol#L587](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587)


## 11. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signitures 


## 12. Setting the constructor to payable

You can cut out 10 opcodes in the creation-time EVM bytecode if you declare a constructor payable. Making the constructor payable eliminates the need for an initial check of msg.value == 0 and saves 13 gas on deployment with no security risks.

- [EthRouter.sol#L90](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90)

- [PrivatePool.sol#L143](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143)


## 13. part of the code can be pre calculated

these parts of the code can be pre calculated and given to the contract as constants this will stop the use of extra operations
even if its for code readability consider putting comments instead.

`18 - 4`
- [PrivatePool.sol#L733](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L733)

`keccak256("ERC3156FlashBorrower.onFlashLoan")`
- [PrivatePool.sol#L642](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L642)


## 14. Use selfbalance() instead of address(this).balance

Use assembly when getting a contract's balance of ETH.

You can use selfbalance() instead of address(this).balance when getting your contract's balance of ETH to save gas. Additionally, you can use balance(address) instead of address.balance() when getting an external contract's balance of ETH.

Saves 15 gas when checking internal balance, 6 for external

- [EthRouter.sol#L141](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L141)
- [EthRouter.sol#L142](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L142)
- [EthRouter.sol#L203](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L203)
- [EthRouter.sol#L208](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L208)
- [EthRouter.sol#L290](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L290)
- [EthRouter.sol#L291](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L291)


## 15. Duplicated require()/revert() checks should be refactored to a modifier or function

Saves deployment costs

```solidity
        if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
```
- [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)

```solidity
    baseToken != address(0)
```
- [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)


## 16. sorting the lines better will result in less storage access needed which result in gas save

of we sort these lines like this less gas will be used for storage access because some of these variables are inside one slot of storage

change this
```solidity
        baseToken = _baseToken;
        nft = _nft;
        virtualBaseTokenReserves = _virtualBaseTokenReserves;
        virtualNftReserves = _virtualNftReserves;
        changeFee = _changeFee;
        feeRate = _feeRate;
        merkleRoot = _merkleRoot;
        useStolenNftOracle = _useStolenNftOracle;
        payRoyalties = _payRoyalties;

        // mark the pool as initialized
        initialized = true;
```
to this
```solidity
        merkleRoot = _merkleRoot;
        virtualBaseTokenReserves = _virtualBaseTokenReserves;
        virtualNftReserves = _virtualNftReserves;
        baseToken = _baseToken;
        nft = _nft;
        changeFee = _changeFee;
        feeRate = _feeRate;
        // mark the pool as initialized
        initialized = true;
        payRoyalties = _payRoyalties;
        useStolenNftOracle = _useStolenNftOracle;
```
(although code looks cleaner if the sorting of declarations changes and `initialized` be the last bool to be made, consider changing it here - [PrivatePool.sol#L94-L100](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L94-L100))

link to the code
- [PrivatePool.sol#L175-L186](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L175-L186)


## 17. cache parts of code for second use

instead of doing some operations again answer can be cached at the first time and be used every where.

cache `(netInputAmount - feeAmount - protocolFeeAmount)` before #L230 and use the cached version in #L236 as well
- [PrivatePool.sol#L230-L236](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230-L236)
