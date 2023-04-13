## Pragma version `^0.8.19` is too recent to be trusted.

Unexpected bugs may come up over time. It's better to choose a lower more battle-tested version. `0.8.10` is recommended.
### Instances
`PrivatePool.sol#L2` `Factory.sol#L2` `EthRouter.sol#L2` `IStolenNftOracle.sol#L2` `PrivatePoolMetadata.sol#L2`

## Use `safeTransfer/safeTransferFrom` instead of `transfer/transferFrom` consistently
`transfer` function may cause silent failures and affect token accounting in the contract.
### Instances
`Factory.sol#L152` `Factory.sol#L115` `Factory.sol#L651` `PrivatePool.sol#L365` `PrivatePool.sol#L527`

## Missing `@return` and `@param` in NatSpec comments
Missing description of `_payRoyalties` parameter
`PrivatePool.sol#L156`
Missing description of `protocolFeeAmount` return var
`PrivatePool.sol#L210` `PrivatePool.sol#L300` `PrivatePool.sol#L697` `PrivatePool.sol#L712`
Missing description of return variables. 
`PrivatePool.sol#L384`

## Any user can withdraw eth stuck in `EthRouter` contract
A user can call `buy()` with an empty `Buy[]` array to drain all eth stuck in EthRouter contract.
### Instances
`EthRouter.sol#L142`

## Tokens accidentally sent to the pool contract cannot be recovered

### Recommendation
Add recovery code: 
``` /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param token is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
  function rescueERC20(address account, address token, uint256 amount) public onlyOwner returns (bool) {
    IERC20(token).transfer(account, amount);
    return true;
  }
} 
```
This does arguably give a lot of power to the owner though since they can basically drain the pool at any moment. Proceed with caution...
## Pool events missing parameters.

Contracts or web pages listening to events cannot react to users' activity since emit does not include the original sender of the transaction.

### Instances
`PrivatePool.sol#L59-L63`

### Recommendation
```
emit Buy(tx.origin, tokenIds, tokenWeights, netInputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);
emit Sell(tx.origin, tokenIds, tokenWeights, netOutputAmount, feeAmount, protocolFeeAmount, royaltyFeeAmount);
emit Change(tx.origin, inputTokenIds, inputTokenWeights, outputTokenIds, outputTokenWeights, feeAmount, protocolFeeAmount);
```
