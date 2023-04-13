     1.Invariant unnecessarily being calculated in every iteration
- Inside sell() function of PrivatePool contract,
- there is need to calculate SalePrice in each iteration of loop as it will remain same during all the iteration in all the scenario.
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L301-L373 
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L335 

      2.Multiple function call within function call can be avoided
- Inside SetAllParamter() function of PrivatePool contract,
- Five different set() functions are being called, which can be avoided, by directly updating the state variables in there, instead of calling each individual function.
- All these functions have onlyOwner modifiers and there is additional upper bound check for setFee(), which can be easily set.
- It will also avoid 4 separate call to onlyOwner modifier, which inturns calls external functions to check NFT ownership to confirm caller is owner.
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L602-L615 


      3.External call every time onlyOwner is called  (which is frequent)
- OnlyOwner modifier has to make an external function call to the NFT factory contract to check whether the msg.sender is owner or not.
- It will be more efficient to set the owner once in state variable and use that to check ownership.
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L127-L132 


      4.Unused parameters passed to function
- A view function flashfee() of PrivatePool contract, takes two argument address and uint256
- But never uses them.
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L750-L752 
