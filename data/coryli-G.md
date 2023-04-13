## Calculations should be cached.
SLOADs are expensive (~100 gas) compared to MLOAD/MSTORE(~3gas).
### Instances
`PrivatePool.sol#L230`, `PrivatePool.sol#L236`, `PrivatePool.sol#L323`, `PrivatePool.sol#L335`

## Using Bools for storage incurs overhead
Use `uint256` for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.
### Instances
`PrivatePool.sol#L94` `PrivatePool.sol#L97` `PrivatePool.sol#L100`

## `abi.encode()` is less efficient than `abi.encodePacked()`
Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see Solidity-Encode-Gas-Comparison).
### Instances
`EthRouter.sol#L177` `EthRouter.sol#L675`

## No need to explicitly initialize variables with default values
If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.
### Instances
`PrivatePool.sol#L237`, `PrivatePool.sol#L328` also every `for` loop counter.

## Increments can be unchecked

### Instances
`PrivatePool.sol#L238` `PrivatePool.sol#L272` `PrivatePool.sol#L329` `PrivatePool.sol#L441` `PrivatePool.sol#L446` `EthRouter.sol#L106` `EthRouter.sol#L116` `EthRouter.sol#L134` `EthRouter.sol#L159`
### Recommendation
```
for (uint256 i; i < tokenIds.length;) {  
 // ...  
 unchecked { ++i; }
}
```

## Public functions to external saves gas
### Instances
`PrivatePool.sol#L167` `PrivatePool.sol#L212` `PrivatePool.sol#L306` `PrivatePool.sol#L393` `PrivatePool.sol#L459` `EthRouter.sol#L99` `EthRouter.sol#L152` `EthRouter.sol#L226` `EthRouter.sol#L254` `EthRouter.sol/L302`
`Factory.sol#L84` `Factory.sol#L129` `Factory.sol#L135` `Factory.sol#L141` `Factory.sol#L148` `Factory.sol#L161`

## Setting the constructor to `payable`
Setting the constructor to `payable` saves ~13 gas per instance.
### Instances
`EthRouter.sol#L90` `Factory.sol#L53` `PrivatePool.sol#L143`
	
## Functions guaranteed to revert when called by normal users can be marked `payable`
If a function modifier such as `onlyOwner` is used, the function will revert if a normal user tries to pay the function. Marking the function as `payable` will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.
### Instances
`Factory.sol#L129` `Factory.sol#L135` `Factory.sol#L141` `Factory.sol#L148`

## `X = X + Y` is cheaper than `X += Y`
### Instances
`PrivatePool.sol#L230` `PrivatePool.sol#L247` `PrivatePool.sol#L252` `PrivatePool.sol#L324` `PrivatePool.sol#L341` `PrivatePool.sol#L678`

## Keccak constant values should be cached instead of computed every time.
### Instances
`PrivatePool.sol#L642`

### Recommendation
```
bytes32 public constant ON_FLASH_LOAN_SELECTOR = keccak256("ERC3156FlashBorrower.onFlashLoan");
```
