`Low Risk Finding/Non Critical ~ 1` 




# Use of Unlocked/Floating Pragma while specifying the complier version

## Summary 
Use of `Floating` or `Unlocked` Pragma eg, `pragma ^0.8.19` Pragma directives with the carrot sign `^`in a codebase allows for testing and deploying with a different complier versions, which can cause undefined behaviour when deployed on the mainet. 

## Vulnerability Details
> Testing & Deploying of Smart Contracts with different complier versions.

The bug arise due to the Carrot sign `^` used in the pragma directive, 
``` solidity
2. pragma solidity ^0.8.19;
```
which allows the contract to be tested with a different complier version from the one that it would be deployed with on the mainet, as one can run tests with a different complier version that has different features and security checks but the newer one that is used for deployment may support a different set of features and security checks, thus this mismatch between the testing and deployment is allowed by the use of this pragma (which can cause the smart contracts to behave in a way unintended by the developers), 
hence is not recommended to be used

## Code Snippet

### Instances - 4

[Factory.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol) `Link to code`
``` solidity 
1. // SPDX-License-Identifier: MIT
2. pragma solidity ^0.8.19;
```

[PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol) `Link to code`
``` solidity 
1. // SPDX-License-Identifier: MIT
2. pragma solidity ^0.8.19;
```
[EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol) `Link to code`
``` solidity 
1. // SPDX-License-Identifier: MIT
2. pragma solidity ^0.8.19;
```
[PrivatePoolMetadata.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol) `Link to code`
``` solidity 
1. // SPDX-License-Identifier: MIT
2. pragma solidity ^0.8.19;
```
## Impact
Use of different complier versions for testing and deployment can enable certain bugs to slip during testing due to the different bugs and bug fixes of different complier versions, which can cause *Undefined Behaviour in the Applications of the Protocol* when deployed on the mainet and also missing checks that could bring about *Over/Under Flow Issues* Which arise due to missing SMT(SafeMaths) checks.

## Tools Use
```
Manual Review & Security Pitfalls & Best Practices (https://youtu.be/IXm6JAprhuw)
```

# Recommendation
**Insure the use of a specific Complier Version**
> Use of Locked Pragma for specifying the complier version to be used for testing and deployment.

``` solidity 
1. // SPDX-License-Identifier: MIT
2. pragma solidity 0.8.19;
```
That is by removing the carrot sign `^`



