## Low and Non-Critical Issues Summary
| Number |Issues Details |
|:--:|:-------|
|[NC-01]| Use stable pragma statement
|[NC-02]| Add more indexed fields to Events for better traceability
|[L-01]| Missing events for important parameter changes
|[L-02]| Possible to pass address 0 in functions that pass ownership
|[L-03]| Empty receive() function in EthRouter.sol
|[L-04]| Function sellQuote has misleading NatSpec
|[L-05]| flashFee function has two input variables which are not used

## |[NC-01]| Use stable pragma statement

Using a floating pragma statement `^0.8.19` is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.

## |[NC-02]| Add more indexed fields to events for better traceability

It is recommended events to have more than one indexed parameters if possible to make searching event logs faster.

Factory.sol:
Add tokenIds to be indexed in the Create event definition
BEFORE:
```
event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);
```
AFTER:
```
event Create(address indexed privatePool, indexed uint256[] tokenIds, uint256 baseTokenAmount);
```

## [L-01] MISSING EVENT FOR IMPORTANT PARAMETER CHANGE

Emitting events allows monitoring activities with off-chain monitoring tools.
I would suggest creating custom events and emitting them on critical parameter changes. I am giving an example for a few files. 

```solidity
Factory.sol
Define custom events:
event PrivatePoolMetaDataSet(indexed address privatePoolMetadata)
event PrivatePoolImplementationSet(indexed address privatePoolImplementation)
event ProtocolFeeRateSet();

function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
        privatePoolMetadata = _privatePoolMetadata;
        emit PrivatePoolMetadataSet(privatePoolMetadata)
    }

function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
        emit PrivatePoolImplementationSet(privatePoolImplementation);
    }

function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
        emit ProtocolFeeRateSet();
    }
```

***

## |[L-02]| Possible to pass address 0 in functions that accept addressed for parameters. These also initialize some contracts or create Private Pools
There are functions that accept an address as a parameter and I recommend to add a check whether someone is passing address 0.

EthRouter.sol
function buy()
We can check if the pool address from the buys array is address 0 and if so - skip any other code execution
```solidity
for (uint256 i = 0; i < buys.length; i++) {
            //Add the address 0 check
            if (buys[i].pool == address(0)) continue;
            if (buys[i].isPublicPool) {
                // execute the buy against a public pool
                uint256 inputAmount = Pair(buys[i].pool).nftBuy{value: buys[i].baseTokenAmount}(
                    buys[i].tokenIds, buys[i].baseTokenAmount, 0
                );
```
function sell()
We can check if sells pool address from the sells array is address 0 and if so - skip any other code execution

```solidity
for (uint256 i = 0; i < sells.length; i++) {
    // Add address 0 check here
    if (sells[i].pool == address(0)) continue;
            // transfer the NFTs into the router from the caller
            for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
                ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);
            }
```
Factory.sol

Here we can pass a _nft address 0 and create a useless Private Pool
```solidity
function create(
        address _baseToken,
        address _nft,
        uint128 _virtualBaseTokenReserves,
        uint128 _virtualNftReserves,
        uint56 _changeFee,
        uint16 _feeRate,
        bytes32 _merkleRoot,
        bool _useStolenNftOracle,
        bool _payRoyalties,
        bytes32 _salt,
        uint256[] memory tokenIds, // put in memory to avoid stack too deep error
        uint256 baseTokenAmount
    ) public payable returns (PrivatePool privatePool)
```
***
## |[L-03]| Empty receive() function in EthRouter.sol
An empty receive function means that the contract can hold Ether and if someone sends
ether to the contract by mistake.
It will be in the contract and when we refund the surplus eth in the buy/change function or at sell
we will actually send the entire Ether balance thus someone will get more eth that they are owed.

function buy() change()
```solidity
if (address(this).balance > 0) {
            msg.sender.safeTransferETH(address(this).balance);
        }
```

function sell()
```solidity
 // transfer the output amount to the caller
msg.sender.safeTransferETH(address(this).balance);
```

One option would be removing the receive() function so no one can send eth by mistake


***
## |[L-04]| Function sellQuote has misleading NatSpec
In NatSpec we see "@return netOutputAmount The output amount of base tokens inclusive of the fee." 
that netOutputAmount should include fees, but it actually does not. Also one of the return variables does not have 
NatSpec protocolFeeAmount

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L711-L723
```solidity
netOutputAmount = outputAmount - feeAmount - protocolFeeAmount;
```

***
## |[L-05]| flashFee function has two input variables which are not used
```solidity
flashFee(address, uint256)
```
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L748-L752
