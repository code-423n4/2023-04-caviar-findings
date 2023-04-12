## [L-01] Users can't create pools while setPrivatePoolImplementation() didn't called by owner.
SC: Factory.sol

## Proof of Concept
Users can't create pools while setPrivatePoolImplementation() didn't called by owner.
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135

## Recommended Mitigation Steps
To avoid situation where users can't create pools it is necessary to set poolImplemmentation in factory constructor.

## [L-02] Unsafe transferOwnership() by solmate.
SC: Factory.sol with imported Owned.sol by solmate lib.

## Proof of Concept
The function transferOwnership() transfer ownership to a new address. In case a wrong address ownership will be permanently lose.


## Recommended Mitigation Steps

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has transferOwnership function, use is more secure due to 2-stage ownership transfer.
![Tux, the Linux mascot](https://i.imgur.com/buDGgQr.png)

## [L-03] deposit() does not has `onlyOwner` modifier.
SC: PrivatePool.sol

## Proof of Concept
Despite the fact that the owner mainly will use the function, it is likely that it will be called by an ordinary user due to the lack of `onlyOwner` modifier. There is no any rewards for whom who made deposit, no LPs, and no withdraw option for users.

## Recommended Mitigation Steps
Add `onlyOwner` modifier.

## [L-04] buy() function will be reverted if virtualNftReserves is equal or less than 1e18.
SC: PrivatePool.sol

## Proof of Concept
Due to the buyQuote() calculation method, tx will be reverted, because outputAmount minimum is 1e18, and if `virtualNftReserves` will be = 1e18, denominator = 0 which lead to revert.
```solidity
        uint256 inputAmount = FixedPointMathLib.mulDivUp(
            outputAmount,
            virtualBaseTokenReserves,
            (virtualNftReserves - outputAmount) //<<-here
        );
```  
Foundry test:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;
import "forge-std/Test.sol";
import "../../src/Factory.sol";
import "../../src/EthRouter.sol";
import "../../src/PrivatePool.sol";
import "../shared/Milady.sol";

contract MyLive is Test {
    address public nftArtist = vm.addr(123);
    address public royaltyRegistry = vm.addr(999);
    address public poolCreator = vm.addr(432);
    address public user2 = vm.addr(4323);

    uint256[] tokenWeights;
    PrivatePool.MerkleMultiProof proofs;

    Factory factory;
    EthRouter ethrouter;
    PrivatePool privatePool;
    Milady milady;

    function setUp() public {
        factory = new Factory();
        ethrouter = new EthRouter(royaltyRegistry); //accept royaltyRegistry addr in constructor;
        milady = new Milady();
        privatePool = new PrivatePool(
            address(factory),
            address(royaltyRegistry),
            address(0)
        ); //Deploy base for clones copy.
        vm.deal(poolCreator, 100 ether);
        vm.deal(user2, 100 ether);
        vm.startPrank(poolCreator);
        milady.mint(poolCreator, 1);
        milady.approve(address(factory), 1);
        vm.stopPrank();
    }

    function testFactory() public {
        console.log("address factory:                   ", address(factory));
        console.log("nftArtist for royalty receive:     ", nftArtist);

        factory.setPrivatePoolImplementation(address(privatePool));

        vm.startPrank(poolCreator);
        uint256[] memory tokenIds = new uint256[](1);
        tokenIds[0] = 1;
        privatePool = factory.create{value: 1 ether}(
            address(0), //_baseToken
            address(milady), //nft
            10e18, //_virtualBaseTokenReserves 
            1e18, //_virtualNftReserves        
            200, //_changeFee
            0, //fee rate 
            bytes32(0), //_merkleRoot
            true, //nft oracle
            false, //pay royalties
            bytes32(address(this).balance + 12345), //salt
            tokenIds, //tokenIds 
            1 ether //baseTokenAmount
        );
        vm.stopPrank();
        vm.startPrank(user2);
        (
            uint256 netInputAmount,
            uint256 feeAmount,
            uint256 protocolFeeAmount
        ) = privatePool.buyQuote(tokenIds.length * 1e18);

        //buy
        (uint256 returnedNetInputAmount, , ) = privatePool.buy{
            value: netInputAmount
        }(tokenIds, tokenWeights, proofs);
    }
}

```
## Recommended Mitigation Steps
In PrivatePool.sol in function`setVirtualReserves()` and factory.sol in `create()` function add
```solidity
require(setVirtualReserves > 1e18);
```

## [L-05] Missing maxInput amount in buy() function can lead to unexpected costs for user.
SC: EthRouter.sol, PrivatePool.sol

## Proof of Concept
This is good practice for AMM to intergrate `maxInput` amount in `buy()` fuction to avoid unexpected costs for user.

Let's see at normal buy case:
1] User chose expensive NFT;
2] User calls `buy()` with more msg.value than `netInputAmount()` require;
3] Rest msg.value comes to user at the end of tx.
Not let's see at another case:
1] User chose NFT;
2] User calls `buy()` with more msg.value than `netInputAmount()` require;
3] Owner of pool see tx in mempool for huge buy() and set `setFeeRate` at 50%
4] User who expected pay 10%, will pay 50% instead.

## Recommended Mitigation Steps
To avoid situation where users can't create pools it is necessary to set poolImplemmentation in factory constructor.

## [L-06] Protocol doesn't support >=36 decimals tokens.
SC: PrivatePool.sol

## Proof of Concept
As we see in function `price()`, if `ERC20(baseToken).decimals()` returned `36`, price will be `0`.
If `ERC20(baseToken).decimals()` returned more than `36` than tx will be reverted.
```solidity
    function price() public view returns (uint256) {
          // ensure that the exponent is always to 18 decimals of accuracy
        uint256 exponent = baseToken == address(0)
            ? 18
            : (36 - ERC20(baseToken).decimals());
        return (virtualBaseTokenReserves * 10 ** exponent) / virtualNftReserves;
    }

```

## Recommended Mitigation Steps
Add `require(ERC20(baseToken).decimals() < 36, "Error decimals")` while creating pool.

## [L-07] PrivatePool buy() function missed implemented require for check array lengths 
SC: PrivatePool.sol

## Proof of Concept
`buy()` function in PrivatePool.sol do not check length of array `tokenIds` and `tokenWeights`.
User could passed different length and tx will be reverted.

## Recommended Mitigation Steps
Add `require(tokenIds.length == tokenWeights.length)`;

## [L-08] PrivatePool sell() function missed implemented require for check array lengths 
SC: PrivatePool.sol

## Proof of Concept
`sell()` function in PrivatePool.sol do not check length of array `tokenIds` and `tokenWeights`.
User could passed different length and tx will be reverted.

## Recommended Mitigation Steps
Add `require(tokenIds.length == tokenWeights.length)`;

## [L-09] PrivatePool change() function missed implemented require for check array lengths 
SC: PrivatePool.sol

## Proof of Concept
`change()` function in PrivatePool.sol do not check length of array `inputTokenIds`, `inputTokenWeights`, outputTokenIds, outputTokenWeights.
User could passed different length and tx will be reverted.

## Recommended Mitigation Steps
Add `require(inputTokenIds.length == inputTokenWeights.length == outputTokenIds.length == outputTokenWeights.length)`;

## [L-10] User could call buy() and do not pass any tokenIds and tx will not be reverted.
SC: PrivatePool.sol

## Proof of Concept
There is no check for `tokenIds` array length in buy() function. User's tx will not be reverted.
Foundry test:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;
import "forge-std/Test.sol";
import "../../src/Factory.sol";
import "../../src/EthRouter.sol";
import "../../src/PrivatePool.sol";
import "../shared/Milady.sol";

contract MyLive is Test {
    address public nftArtist = vm.addr(123);
    address public royaltyRegistry = vm.addr(999);
    address public poolCreator = vm.addr(432);
    address public user2 = vm.addr(4323);

    uint256[] tokenWeights;
    PrivatePool.MerkleMultiProof proofs;
    IStolenNftOracle.Message[] stolenNftProofs;

    Factory factory;
    EthRouter ethrouter;
    PrivatePool privatePool;
    Milady milady;

    function setUp() public {
        factory = new Factory();
        ethrouter = new EthRouter(royaltyRegistry); //accept royaltyRegistry addr in constructor;
        milady = new Milady();
        privatePool = new PrivatePool(
            address(factory),
            address(royaltyRegistry),
            address(0)
        ); //Deploy base for clones copy.
        vm.deal(poolCreator, 100 ether);
        vm.deal(user2, 100 ether);
        vm.startPrank(poolCreator);
        milady.mint(poolCreator, 1);
        milady.approve(address(factory), 1);
        vm.stopPrank();
    }

    function testDirectSell() public {
        console.log("address factory:                   ", address(factory));

        factory.setPrivatePoolImplementation(address(privatePool));

        vm.startPrank(poolCreator);
        uint256[] memory tokenIds = new uint256[](1);
        tokenIds[0] = 1;
        privatePool = factory.create{value: 1 ether}(
            address(0), //_baseToken
            address(milady), //nft
            10e18, //_virtualBaseTokenReserves
            2e18, //_virtualNftReserves
            0, //_changeFee
            0, //fee rate //@note покрутиьт потом с разными значениями
            bytes32(0), //_merkleRoot
            false, //nft oracle
            false, //pay royalties
            bytes32(address(this).balance + 12345), //salt
            tokenIds, //tokenIds //@audit Poll could be created with 0 nft and msg.value>0; check what could be
            1 ether //baseTokenAmount
        );
        vm.stopPrank();

        console.log("======================BEFORE BUY======================");
        console.log("User2 eth balance                  ", user2.balance);
        console.log(
            "Pool eth balance                   ",
            address(privatePool).balance
        );
        console.log(
            "Pool nft balance                   ",
            milady.balanceOf(address(privatePool))
        );

        vm.startPrank(user2);
        (
            uint256 netInputAmount,
            uint256 feeAmount,
            uint256 protocolFeeAmount
        ) = privatePool.buyQuote(tokenIds.length * 1e18);

        //buy
        (uint256 returnedNetInputAmount, , ) = privatePool.buy{
            value: netInputAmount
        }(tokenIds, tokenWeights, proofs);
        console.log("=================AFTER BUY & BEFORE SELL=============");
        console.log("User2 eth balance                  ", user2.balance);
        console.log(
            "Pool eth balance                   ",
            address(privatePool).balance
        );
        console.log(
            "Pool nft balance                   ",
            milady.balanceOf(address(privatePool))
        );

        (uint256 netOutputAmount, , ) = privatePool.sellQuote(
            tokenIds.length * 1e18
        );
        uint256 balanceBefore = address(this).balance;

        //sell
        milady.approve(address(privatePool), 1);
        privatePool.sell(tokenIds, tokenWeights, proofs, stolenNftProofs);
        console.log("======================AFTER SELL======================");
        console.log("User2 eth balance                  ", user2.balance);
        console.log(
            "Pool eth balance                   ",
            address(privatePool).balance
        );
        console.log(
            "Pool nft balance                   ",
            milady.balanceOf(address(privatePool))
        );

        //sellE
        uint256[] memory tokenIdsE;
        privatePool.sell(tokenIdsE, tokenWeights, proofs, stolenNftProofs);
        console.log("======================AFTER SELL E====================");
        console.log("User2 eth balance                  ", user2.balance);
        console.log(
            "Pool eth balance                   ",
            address(privatePool).balance
        );
        console.log(
            "Pool nft balance                   ",
            milady.balanceOf(address(privatePool))
        );
    }
}

```

## Recommended Mitigation Steps
Add `require(tokenIds.length > 0)`;

## [L-11] User could call sell() and do not pass any tokenIds and tx will not be reverted.
SC: PrivatePool.sol

## Proof of Concept
There is no check for `tokenIds` array length in sell() function. User's tx will not be reverted.


## Recommended Mitigation Steps
Add `require(tokenIds.length > 0)`;

## [L-11] User could call deposit() and do not pass any tokenIds and tx will not be reverted.
SC: PrivatePool.sol

## Proof of Concept
There is no check for `tokenIds` array length in deposit() function. User's tx will not be reverted.
Also event `Deposit()` will be broadcasted.


## Recommended Mitigation Steps
Add `require(tokenIds.length > 0)`;