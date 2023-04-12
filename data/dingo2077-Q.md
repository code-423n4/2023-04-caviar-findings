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