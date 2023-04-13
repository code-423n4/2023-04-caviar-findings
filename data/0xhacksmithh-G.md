### [Gas-01] A no. of ways present to `Short-circuit` Function call which significantly save high gas
Like in ```buy()```
*First way*
Is to move ```if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount(); ``` to first step of function
If any user send any ETH(when basetoken is not ETH) then this function will revert right after, So no further gas wasted in further calculation

*Second way* 
To add a check whether both input array are of same size or not. 

*Third way*
Gas can save in ```buy()``` by simply checking user input ids are owned by pool or not
buy() simply makes all weight calculation, price calculation, then update state variables and then try to send nfts to caller.
If anyone of nfts not owned by pool then whole call will failed, and all gas which used to calculate above things will wasted.
So if you move important checks ```like is all nft ids input by caller is owned by protocal``` to the above and then proceed with weigth, price etc, it will cost less gas if any user intentionaly/unintensionaly input wrong ids.
```solidity
function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof) // @audit-issue function expose to reentrancy
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount) // @audit much gas can save by making initial check whether these nft own by this contract or not, Save caller gas in case of front-running
    {
        // ~~~ Checks ~~~ // // @audit short-circuiting by checking if length of Ids and Weights are same or not.
+       if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount(); 
+       require(tokenIds.length == tokenWeights.length);
        // calculate the sum of weights of the NFTs to buy
        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

        // calculate the required net input amount and fee amount
        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

        // check that the caller sent 0 ETH if the base token is not ETH // @audit this can preform earlier to save gas
-        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();  // @audit use assembly

        // ~~~ Effects ~~~ //
```
```
File: src/PrivatePool.sol
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L289
```


### [Gas-02] With `assembly` ETH Can Be Send To Save Gas
`return` data(bool success, ) has to be stored due to EVM architechture, but in a usage like below, `out` and `outsize` values are given (0,0) this storage disappears and gas optimization is provided.

```solidity
(bool succ, ) = dest.call{value: amount}("");
+ bool succ;
+ assembly{
         succ := call(gas(),dest,amount,0,0)
+    }
+ require(succ); 
```

*Instances(Multiple instances)*
```
File : src/PrivatePool.sol

```

### [Gas-03] Avoid Compound Assignment Operation In `state` Variable
Using compound assignment operators for state variable(like state += x, or state -= x) is more expensive than using operator assignment(like state = state + x, or state = state - x).

*Instances(3)*
```
File : src/PrivatePool.sol
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L252
```
### [Gas-04] `keccak256()` Should Only Need To Be Called On A Specific String Literal Only Once
Should be store in a `immutable` state variable, and called from there when required.

*Instances(1)*
```
File : src/PrivatePool.sol
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L642
```

### [Gas-05] Use `selfbalance()` Instead Of `address(this).balance`

*Instances(Multiple Instances)*
```
File : src/PrivatePool.sol
```



