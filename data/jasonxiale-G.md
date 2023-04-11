# no need to copy calldata into memory
## since we're only reading form __changes__, there is no need to copy calldata into memory
File src/EthRouter.sol#L262

            Change memory _change = changes[i];

# State variables only set in the constructor should be declared immutable
File src/PrivatePool.sol

    82    address public baseToken;
    85    address public nft;
    88    uint56 public changeFee;
    94    bool public initialized;

# <x> += <y> costs more gas than <x> = <x> + <y> for state variables
File src/PrivatePool.sol

    230    virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
    231    virtualNftReserves -= uint128(weightSum);
    323    virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount);
    324    virtualNftReserves += uint128(weightSum);