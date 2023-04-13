## > 0 is less efficient than != 0 for unsigned integers
Recommended to use the != 0 operator instead of the > 0 operator when comparing unsigned integers to zero in Solidity, as it can help to optimize gas costs and contract execution time.

https://code4rena.com/reports/2022-03-timeswap/#low-risk-and-non-critical-issues 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L87 
       if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 
       0)) { 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L121

                       if (royaltyFee > 0) { 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L141 

        if (address(this).balance > 0) { 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L188 

                        if (royaltyFee > 0) {

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L290 
                   if (address(this).balance > 0) { 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol

## Use custom errors to save deployment and runtime costs in case of revert
Instead of using strings for error messages, you can use custom errors to reduce both deployment and runtime gas costs. In addition, they are very convenient as you can easily pass dynamic information to them.


https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L76

https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L100 

https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L114 
https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L212 

https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L228 

https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L273

## Use solidity version 0.8.19 to gain some gas boost
Upgrade to the latest solidity version 0.8.19 to get additional gas savings.

See the latest release for reference: https://blog.soliditylang.org/2023/02/22/solidity-0.8.19-release-announcement/

Proof Of Concept
https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L2

## <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L230 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L247 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L252

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L324 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L341 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L678


https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L184 
https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L431 
https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol#L560 

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L231 
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L355
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L323

## Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address

References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

https://twitter.com/transmissions11/status/1518507047943245824

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol
https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/utilities/poolsUtility/Position.sol
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol
https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol

