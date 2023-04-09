**src/EthRouter.sol**
- L99/152/219/254 - Multiple functions are defined as public but are not used within the contract, so they should be defined as external or whatever, as appropriate.

- L115/182 - A division is made with buys[i].tokenIds.length and sells[i].tokenIds.length, this value comes from a function input that is public, therefore, this should be validated with a require() or if() and revert() that is not zero, since it would throw an exception.

- L101/154/228/256 - The deadline that is received as input is validated but it is not used for anything inside the function, therefore, it is not clear why it should be there.
 
- L314 - When it is validated if royaltyFee > salePrice, it is left available so that it does not revert if royaltyFee == salePrice, it seems to me that this should also be validated since if I sell something and it does not leave me profits, I would not want to sell it. Therefore, the correct validation should be: if (royaltyFee >= salePrice).


**src/PrivatePool.sol**
- L134/143/623 - Function ordering does not follow the Solidity style guide
According to the Solidity style guide, functions should be laid out in the following order :constructor(), receive(), fallback(), external, public, internal, private, but the cases below do not follow this pattern.

- L236 - A division is made by the input tokenIds.length, but it is never checked if it is != 0, this should be validated with a require.

- L791 - When it is validated if royaltyFee > salePrice, it is left available so that it does not revert if royaltyFee == salePrice, it seems to me that this should also be validated because if I sell something and it does not leave me benefits, I would not want to sell it. Therefore, the correct validation should be: if (royaltyFee >= salePrice).

- L82/88/750/755 - The variables in memory baseToken and changeFee are public, therefore they have their own getter function, but also the public functions flashFeeToken() and flashFee() are created that return the same value. Therefore, this can cause a lot of confusion. The two possible options would be to either remove the flashFeeToken() or flashFee() functions or make the in-memory variables baseToken and changeFee private.

- L157/211/301/385/459/484/514/742/750/755 - Multiple functions are public but are never used within the contract, therefore they should be defined as externals.

- L178/541/745 - The virtual memory variable NftReserves can have multiple values, including zero. In line 745 a division is made by this value, since zero is a possible value it should be validated that it does not have that defined value. This should be validated with a require().

- L673/675 - Two arrays are traversed, iterating only one, without validating if they have the same length. This can be a problem, as it could be reversed because tokenWeights.length < tokenIds.length or cause strange behavior if tokenWeights.length > tokenIds.length.

- L172/177/178/179/180/181/182/183/602 - Instead of making all the settings in the initialize() function, you can directly call the setAllParameters() function, this would perform the validations without having to duplicate them.