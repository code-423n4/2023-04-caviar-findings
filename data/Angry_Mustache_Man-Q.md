## [L-1] Division before multiplication causing significant loss of precision
Calculations in the code first divides and then multiplies again, there is a significant loss of precision

### Recommended Mitigation Steps

Multiply first before dividing to keep the precision.
## [L-2] FLOATING PRAGMA
Contracts
should be deployed with the same compiler version and flags that they
 have been tested with thoroughly. Locking the pragma helps to ensure
 that contracts do not accidentally get deployed using another pragma,
for example, either an outdated pragma version that might introduce bugs
that affect the contract system negatively or a recently released pragma
 version which has not been extensively tested.
### Occurence
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L2

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L2

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L2
## [L-3] Wrong execution
### Impact
Might lead to inappropriate reverts or lock of funds
### Proof of Concept
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L412
The comment above says : // check that the input weights are greater than or equal to the output weights
And also in @notice it is given as : /// @notice Changes a set of NFTs that the caller owns for another set of NFTs in the pool. The caller must approve
/// the pool to transfer the NFTs. The sum of the caller's NFT weights must be less than or equal to the sum of the
/// output pool NFTs weights. The caller must also pay a fee depending the net input weight and change fee amount.
But in code :
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L413

 if (inputWeightSum < outputWeightSum) revert InsufficientInputWeight();

The condition inputWeightSum==outputWeightSum is ommitted 

### Recommendation
Change condition to :
 if (inputWeightSum <= outputWeightSum) revert InsufficientInputWeight();


## [NC-1] Typos in code
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L105
Here,the comment is //loop through and execute the the buys 
using 'the' 2 times which is not needed.