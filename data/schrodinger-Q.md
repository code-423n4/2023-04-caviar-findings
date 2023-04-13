### Low Risk Issues
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | lack of check of array.length of `inputTokens` and `outputTokens` | 1 |
| Total Low Risk Issues | 1 | 
|:--:|:--:|

### [L-01] lack of check of array.length of `inputTokens` and `outputTokens`
the following function dosent check the length of arrays of the input params which might lead to a silent revert if theyre not equal
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L385
```solidity
    function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {
```
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211
```solidity
 function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
    {
```
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L301
```solidity
     function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
```

### Non Critical Issues
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [NC-01] | unnecessary function being invoked | 1 |

| Total Non Critical Issues | 1 | 
|:--:|:--:|
### [NC-01] unnecessary function being invoked
In the function `flashloan` variable fee calls other function `flashfee` which is always constant thats equal to changeFee 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L632

```solidity
 632:  uint256 fee = flashFee(token, tokenId);
```
consider adding changeFee directly instead

```solidity
    uint256 fee = changeFee;
```
