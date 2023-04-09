**src/EthRouter.sol**
- L115/116/134/161/162/166/168/170/171/174/177/182/183/186/196/197 - The variable buys[i].tokenIds.length/sells[ is used multiple times i].tokenIds.length/sells[i], this could be put directly on Line 107 and gas would be saved.

- L141/142/203/208/290/291 - The query to the address(this).balance function is used twice, this could save gas by creating a variable in memory and using that value.

- L101/154/228/256 - The same validation is performed multiple times, this could be unified in an internal function or in a modifier and used by the functions.
 
- L262/265/284 - Instead of using changes[i].inputTokenIds.length, use _change.inputTokenIds.length, this would generate a lower gas cost.


**src/PrivatePool.sol**
- L225 - The validation that does not need any input should go first in the code, that is, in line 219, since if it does not comply, it would be reversed without calling any other function (which is what is done in lines 219/222) .

- L474 - A revert() is performed but no message is defined, this should be added as it raises the level of understanding for the user.
