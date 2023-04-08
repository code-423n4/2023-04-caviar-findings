For all `for` loops, use prefix increment `++i` notation instead of postfix `i++`.

Although minimal difference is observed, it is good practice to reduce as much as possible the amount of gas used if achieving the same functionality.

### POC

```solidity

contract One{
   uint256 public number;
   //Gas used : 43634
   function incrementByOne() public returns (uint256){
       return number++;
   }
}

contract Two{
   uint256 public number;
   //Gas used : 43628
   function incrementByOne() public returns (uint256){
       return ++number;
   }
}
```

Source: [Link](https://dev.to/jamiescript/gas-saving-techniques-in-solidity-324c#incrementing)