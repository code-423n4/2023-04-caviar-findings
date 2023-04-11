# Use `calldata` instead of `memory`

Factory.sol L82

```
function create(
...
uint256[] memory tokenIds // memory -> calldata
...
) 
```