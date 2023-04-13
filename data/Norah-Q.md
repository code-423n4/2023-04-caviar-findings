
      1.No upper bound on protocol fee rate
- In factory contracts setProtocolFeeRate() there is no upper bound for fee-rate like it is in case of setFeeRate() (50% cap).
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141-L143
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562-L571 


       2. Use indexed events
- Consider using indexed events for the parameters like OutputAmount, tokenIdSold etc in Buy,Sell,Change, Deposit and withdraw event.
As they lead to better filtering and better overall tracking
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L59-L63 


      3.No upper bound on changeFee 
- In Private Pool contracts there is no upper bound for changeFee like it is in case of feeRate.
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157-L200 


      4. Return proper reason for revert incase of StolenNFT sell attempt
- Sell() function incase of privatePool contract reverts if user tries to sell StolenNFT
But, it does not provide Custom Error or Revert message doing so.
- It may confuse the user who herself is not aware of NFT being stolen and that is why her transaction is reverting.
- Recommend to either return cutomError like StolenNFTSellAtempt() or revert with message like “Can't sell Stolen NFT”.
- https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L316-L319 
