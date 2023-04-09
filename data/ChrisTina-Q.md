# Function change() in EthRouter.sol reverts if user wants to make several changes (changes.length > 1)

## Affected function
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L261-L273

## Description
The function change() takes as a parameter an array Change[] calldata changes.  
For each element of the input array, the change method of the PrivatePool contract is called, and ETH with a value of msg.value sent along. If the amount sent is not used up, the pool contract refunds the remainder to the EthRouter. 
After the first iteration, the remainder of the funds is held by the EthRouter contract. Because the initial EthRouter balance is 0 and fees have been deducted, the balance of the EthRouter is smaller than msg.value and the second loop iteration will fail with an EVM error (Out of Fund).

## Impact
State changes are reverted, but user incurs gas costs.

## Proof of Concept
Test file available here (in .txt format, change to .sol to run test cases locally) : https://gateway.pinata.cloud/ipfs/QmQrCaJD36UAr1Xqo2AQfbjGVA9GmG3cUqSdxXmP9FgxQN
The tests shows that calling the change() function works fine with an input array of length 1 and fails with an array of input length 2, even if extra ETH is sent along.

## Recommended Mitigation Steps
Only send the amount needed to pay the fees on each iteration.