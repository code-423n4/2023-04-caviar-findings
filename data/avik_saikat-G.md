# array.length inside loop
## Vulnerability details
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

You save `3 gas` by not reading array.length - `3 gas` per instance - `27 gas` saved


## Lines of code
### Total 23
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L106
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L115
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L116
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L134
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L159
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L161
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L182
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L183
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L239
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L261
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L265
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L284
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L119
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L236
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L272
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L446
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L496
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L518
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L673

## Impact
You save `3 gas` by not reading array.length - `3 gas` per instance - `27 gas` saved.

## Recommended Mitigation Steps
- `Cache Array Length Outside of Loop`

``` solidity
// before
for (uint256 i = 0; i < array.length; i++) {
    // do something
}

// after
uint256 len = array.length
for (uint256 i = 0; i < len; i++) {
    // do something
}
```


# Default assignment
## Vulnerability details

Uninitialized variables are assigned with the types default value.

Explicitly initializing a variable with it's default value costs unnecessary gas.

## Lines of code
### Total 21
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L106
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L116
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L134
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L159
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L161
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L183
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L239
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L261
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L265
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L284
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L119
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L237
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L272
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L328
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L446
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L496
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L518
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L673

## Impact
Declaring `uint256 i = 0;` means doing an `MSTORE` of the value `0` Instead you could just declare `uint256 i` to declare the variable without assigning it’s default value, saving `3 gas` per declaration


## Recommended Mitigation Steps

### Example

🤦 Bad:
```solidity
uint256 x = 0;
bool y = false;
```

🚀 Good:
```solidity
uint256 x;
bool y;
```


# Check msg.value first
## Vulnerability details
Reading `msg.value` costs `2 gas`, while reading from memory `costs 3`

## Lines of code 
### Total -> 5
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635

## Impact
Reading msg.value costs 2 gas, while reading from memory costs 3, this will save 1 gas with no downside

## Recommended Mitigation Steps
This will save `1 gas` with no downside

```solidity
//before
if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

// after
if (msg.value < fee && baseToken == address(0)) revert InvalidEthAmount();
```


# Use unchecked for loop counters
## Vulnerability details
You can get cheaper for loops (at least `25 gas`, however can be up to `80 gas` under certain conditions)

## Lines of code
### Total -> 19
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L106  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L116  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L134  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L159  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L161  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L183  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L239  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L261  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L265  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L284  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L119  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L238  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L272  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L329  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L441  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L446  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L496  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L518  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L673
## Impact
Costs gas

## Recommended Mitigation Steps
```solidity
unint256 len = tokenIds.length;
for (uint256 i = 0; i < len; /** NOTE: Removed i++ **/ ) { 
	// Do the thing 
	// Unchecked pre-increment is cheapest 
	unchecked { ++i; } 
}
```


# Use assembly to check for address(0)
## Vulnerability details
You can save 16000 gas per instance in deploying contract.

You can save about 6 gas per instance if using assembly to check for address (0)
## Lines of code
### Total -> 21
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L110  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L149  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L254  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L277  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L278  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L344  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L357  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L421  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L500  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L522  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L651  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L733  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L744  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L47  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L103

## Impact
You can save `16000 gas` per instance in deploying contract.

You can save about `6 gas` per instance if using assembly to check for `address (0)`

## POC
```solidity


contract GasTest is DSTest {
    Contract0 c0;
    Contract1 c1;

    function setUp() public {
        c0 = new Contract0();
        c1 = new Contract1();
    }

    function testGas() public view {
        c0.ownerNotZero(address(this));
        c1.assemblyOwnerNotZero(address(this));
    }
}

contract Contract0 {
    function ownerNotZero(address _addr) public pure {
        require(_addr != address(0), "zero address)");
    }
}

contract Contract1 {
    function assemblyOwnerNotZero(address _addr) public pure {
        assembly {
            if iszero(_addr) {
                mstore(0x00, "zero address")
                revert(0x00, 0x20)
            }
        }
    }
}


```

### Gas Report
```js
╭────────────────────┬─────────────────┬─────┬────────┬─────┬─────────╮
│ Contract0 contract ┆                 ┆     ┆        ┆     ┆         │
╞════════════════════╪═════════════════╪═════╪════════╪═════╪═════════╡
│ Deployment Cost    ┆ Deployment Size ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 61311              ┆ 338             ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ Function Name      ┆ min             ┆ avg ┆ median ┆ max ┆ # calls │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ ownerNotZero       ┆ 258             ┆ 258 ┆ 258    ┆ 258 ┆ 1       │
╰────────────────────┴─────────────────┴─────┴────────┴─────┴─────────╯
╭──────────────────────┬─────────────────┬─────┬────────┬─────┬─────────╮
│ Contract1 contract   ┆                 ┆     ┆        ┆     ┆         │
╞══════════════════════╪═════════════════╪═════╪════════╪═════╪═════════╡
│ Deployment Cost      ┆ Deployment Size ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ 44893                ┆ 255             ┆     ┆        ┆     ┆         │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ Function Name        ┆ min             ┆ avg ┆ median ┆ max ┆ # calls │
├╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌┼╌╌╌╌╌┼╌╌╌╌╌╌╌╌╌┤
│ assemblyOwnerNotZero ┆ 252             ┆ 252 ┆ 252    ┆ 252 ┆ 1       │
╰──────────────────────┴─────────────────┴─────┴────────┴─────┴─────────╯
```


## Recommended Mitigation Steps
```solidity
// before
function ownerNotZero(address _addr) public pure {
    require(_addr != address(0), "zero address)");
}

// after
function assemblyOwnerNotZero(address _addr) public pure {
    assembly {
        if iszero(_addr) {
            mstore(0x00, "zero address")
            revert(0x00, 0x20)
        }
    }
```

# Use x=x+y instad of x+=y
## Vulnerability details
You can save about 35 gas per instance if you change from ``x+=y` to `x=x+y`

This works equally well for subtraction, multiplication and division.
## Lines of code
### Total -> 9
- https://github.com/code-423n4/2023-04-caviar/blob/mai[n/src/PrivatePool.sol#L230  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L247  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L252  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L341  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L678](<- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L247
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L252
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L341
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L355
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L678>)

## Impact
You can save about `35 gas` per instance

## Recommended Mitigation Steps
```solidity
// before
virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);

virtualNftReserves += uint128(weightSum);

//after

virtualBaseTokenReserves =  virtualBaseTokenReserves - uint128(netOutputAmount + protocolFeeAmount + feeAmount);

virtualNftReserves = virtualNftReserves + uint128(weightSum);
```

# Use assembly balance ETH
## Vulnerability details
Use assembly to check for balance of address

## Lines of code
### Total -> 6
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L141  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L142  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L203  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L208  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L290  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L291

## Impact
You can save `159 gas` per instance if using assembly to check for balance of address

## Recommended Mitigation Steps
```solidity
// before
function addressInternalBalance() public returns (uint256) {
        return address(this).balance;
}

// after
returns (uint256) {
  assembly {
      let c := selfbalance()
      mstore(0x00, c)
      return(0x00, 0x20)
  }
}

// before
function addressExternalBalance(address addr) public {
      uint256 bal = address(addr).balance;
}
   
// after
function assemblyExternalBalance(address addr) public {
    uint256 bal;
    assembly {
        bal := balance(addr)
    }
}
```


# Change `public` functions to `external`
## Vulnerability details
External call cost is less expensive than of public functions.  
Contracts [are allowed](https://docs.soliditylang.org/en/latest/contracts.html#function-overriding) to override their parents’ functions and change the visibility from `external` to `public`.
## Lines of code
### Total -> 32
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L99  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L152  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L226  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L254  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L84  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L135  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L148  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L161  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L168  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L167  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L306  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L393  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L459  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L484  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L514  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L538  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L550  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L576  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L587  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L609  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L665  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L731  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L742  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L750  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L755  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L763  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L35  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L55


## Recommended Mitigation Steps
The following functions could be set `external` to save gas and improve code quality:

```solidity
// before
function execute(address target, bytes memory data) public payable onlyOwner returns (bytes memory) {
	// do the thing
}

function execute(address target, bytes memory data) external payable onlyOwner returns (bytes memory) {
	//do the thing
}
```

# `abi.encode()` is less efficient than `abi.encodePacked()`

## Vulnerability details
Changing `abi.encode` to `abi.encodePacked` can save gas. `abi.encode` pads extra null bytes at the end of the call data which is normally unnecessary. In general, `abi.encodePacked` is more gas-efficient.

## Lines of code
### Total -> 2
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L177  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L675

## Impact
Costs more gas
## Recommended Mitigation Steps
Changing `abi.encode` to `abi.encodePacked` can save gas. `abi.encode` pads extra null bytes at the end of the call data which is normally unnecessary. In general, `abi.encodePacked` is more gas-efficient.

```solidity
//before
leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

//after
leafs[i] = keccak256(bytes.concat(keccak256(abi.encodePacked(tokenIds[i], tokenWeights[i]))));
```

# `internal` functions only called once can be inlined to save gas  
  
## Vulnerability details  
  
`Internal/Private` functions only called once can be inlined to save gas  
## Lines of code  
  
### Total -> 2
  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L779  
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112  
  
## Impact  
Not inlining costs `20` to `40 gas` because of two extra `JUMP` instructions and additional stack operations needed for function calls.  
  
## Recommended Mitigation Steps  
  
`Internal/Private` functions only called once can be inlined to save gas

