# Have not check whether the length of input parameter is equal
These following fucntions have not check whether the length of input parameter is equal.It will set an error `emit` if the length of `tokenWeights` is higher than `tokenIds`.

EthRouter.buy, Line 99
EthRouter.sell, Line 301
EthRouter.change, Line 385

# Anyone can use the ETH which is left in the EthRouter
In `EthRouter` contract, anyone can use the ETH which is left in it. Once the `EthRouter` own the ETH, an attacker could call the `EthRouter.buy` function with empty `buys` parameter to gain all the ETH of `EthRouter` or call the `EthRouter.buy` function to buy some NFT which using the ETH of `EthRouter`.