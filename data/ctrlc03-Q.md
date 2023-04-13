# Leftover Ether in the EthRouter contract can be stolen by any user (NC)

Users interacting with a private using the `EthRouter` contract will be refunded any Ether that is present within the contract after calls to the `change`, `sell` or `buy` functions. 

Given that the contract implements a `receive` payable function, funds can be sent by mistake to the contract. 

It is recommended that instead of sending back the full balance of the smart contract to the user interacting with the router, instead the amount due is sent back to the user, and the leftover is sent to the protocol address. Based on the `PrivatePool` code logic, funds could be sent to the `Factory` contract, where they can be withdrawn by the protocol admins.

# Users could overpay fees on flashloan (NC)

When the base token is the native token (`address(0)`), the `msg.value` is checked to be at least the required one for the transaction. However, it could happen that a user sends by mistake a larger `msg.value` and the native token would be lost by the user.

Code: https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635

It is recommended that `msg.value` is checked to be the exact amount expected:

` if (baseToken == address(0) && msg.value != fee) revert InvalidEthAmount();`
