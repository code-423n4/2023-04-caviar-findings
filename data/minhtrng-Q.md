## [01] missing check for msg.value if baseToken not ETH

`PrivatePool.flashLoan` does not check if ETH has been sent, even though baseToken is not ETH. The functions `buy`, `change`, `deposit` do perform this check:

```js
if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
```

This could lead to users sending ETH to the contract on accident. 

Mitigation: perform the check

## [02] underflow for tokens with less than 4 decimals

Function `PrivatePool.changeFeeQuote` will revert if the baseToken has less than 4 decimals, rendering `change` function unusable:

```js
function changeFeeQuote(uint256 inputAmount) public view returns (uint256 feeAmount, uint256 protocolFeeAmount) {

uint256 exponent = baseToken == address(0) ? 18 - 4 : ERC20(baseToken).decimals() - 4;
```

Tokens with less than 4 decimals are too rare for this to likely be an issue, but it could be fixed by adjusting the math (dividing by `10 ** (4-token.decimals)` if `decimals <= 3`).

## [03] owner can do MEV at cost of users

The functions `setFeeRate` and `setVirtualReserves` can be used by the owner to do MEV by frontrunning user transactions and adjusting the values to reach the users max slippage values. 

Only viable mitigation would be removing them. But slippage is something that a user has to take into account anyway, so only a minor issue and likely users responsibility to use periphery properly.

## [04] Documentation-implementation-mismatch for deposits and LPs

According to https://docs.caviar.sh/: `"Custom Pools: [...] Like shared pools, liquidity providers earn fees from trades against their pool."` 

This is not the case in the current implementation (by design) and users might deposit in the believe of earning yield and being able to redeem, but would lose their deposits instead. Mitigation: update docs

## [05] Function documentation misleading for `EthRouter.buy`

`EthRouter.buy` states:

```js
/// @param payRoyalties Whether to pay royalties or not.
```

It might not be clear to users that private pools can not be opted out and the decision to pay royalties is taken by the pools owner. For a better user experience, consider reflecting this in the function documentation.

## [06] obsolete comment

The function `PrivatePool.deposit` can not incur any slippage since it is not designed to mint LPs. Hence the following function documentation is obsolete:

```js
/// @dev DO NOT call this function directly unless you know what you are doing. Instead, use a wrapper contract that
/// will check the current price is within the desired bounds.
```