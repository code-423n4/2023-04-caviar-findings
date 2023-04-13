 #### NC-1 Unlocked Pragma

-Description => Some Contracts in scope  make use of floating pragma e.g ^0.8.19

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L2 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L2 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L2

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L2

-Impact => This can lead to problems with different versions being used for testing, production etc as a floating pragma implies any within range may be used could lead to outdated compiler, versions with different bugs etc
-POC => SWC103 https://swcregistry.io/docs/SWC-103
-Tools Used => Manual
-Recommendation => It is recommended to lock the pragma. Make use of pragma solidity 0.8.17 remove caret

#### NC-2 Solidity Version Too Recent

-Description => Current version 0.8.19 is the latest version

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L2 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L2 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L2

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L2

-Impact => Latest version may not have yet identified or fixed bugs. It is recommended to use latest version but not the current lastest one
-POC => Best practise 
-Tools Used => Manual
-Recommendation => It is recommended to use latest versions of solidity e.g 0.8.17 0.8.18 before 0.8.19

#### NC-3 Natspec Code Commenting

-Description => some code in scope does not make use of complete and rich consistent detailed Natspec commenting especially for functions with parameters e.g leaving out a @params 

for function below @param address nft has been ignored in NatSpec 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L301

for function below @param bool _payRoyalties. has been ignored in Natspec
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L71

for function below @param _changeFee and @param _payRoyalties ignored in Natspec 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L157

for function below @return protocolFeeAmount is ignored in Natspec 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211

for function below @param protocolFeeAmount is ignored in Natspec 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L300

for function below @return feeAmount and @return protocolFeeAmount ignored in Natspec
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L385

for function below @return protocolFeeAmount is ignored in Natspec  
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L694

for function below @return protocolFeeAmount is ignored in Natspec  
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L713

Natspec does not specify @return [an address]
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L754

Natspec does not specify comment on @return [a string] 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L35

Natspec does not specify comment on @return [bytes]
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L55

This function does not completely have complete and rich Natspec code commenting 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L55

-Impact => This can reduce the ability or other devs, users, auditors to understand the functions
-POC => https://docs.soliditylang.org/en/v0.8.17/natspec-format.html
-Tools Used => Manual
-Recommendation => It is recommended to fix all relevant function instances to make use of rich Natspec code commenting which allows for solid documentation of functions, how they work, its paramaters and return values to help with code readability and maintainability. All params and returns detailed for all functions. 


#### NC-4 Function Parameters Use of UnderScore

-Description => Some contracts in scope  make use _ underscore in front of function parameters or back  or none . However certain instances or codes and functions as are not consistent with what majority of code 

EthRouter.sol does not use _ underscore in front of function arguments or return values 

Function below some arguments use _ and some do not e.g _salt vs baseTokenAmount
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L71

Factory.sol function setPrivatePoolMetadata, setPrivatePoolImplementation, setProtocolFeeRate use _ but the remaining from withdraw do not
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L148 

PrivatePool function before function below use _ undescore 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L211 

Wheres function below one aragument _nft uses underscore the others dont 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L514 

PrivatePoolMetadata.sol functions don't make use of underscore 

-Impact => This can lead to inconsistencies in code
-POC => Inconsistency
-Tools Used => Manual
-Recommendation => It is recommended to be consistent in functions or codebase unless specific reasons are clear why come arguments may have _ underscore and others not


 #### NC-5 Undescore internal functions 

-Description => Some internal functions in code have undescore e.g _functionInternal whereas others do not 

Should be _trait(
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePoolMetadata.sol#L112

This function has underscore which is ideal
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L778 

#### NC-6 May be ideal and use named return variables throughout code 

-Description => Some Contracts in scope  make use of named return values e.g returns( bool active) however in other functions named returns are not used. 

EthRouter.sol has named return variables 

Factory.sol has named return variables expect for function below 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L161 

PrivatePool.sol most functions have named return values but the following is an example of many without
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L459 

PrivatePoolMetadata.sol contracts do not use named return variables

-Impact => This can lead to confusion on code readability, auditing, maintain, chance to return wrong values or forget to return right values and variables etc
-POC => Best practise
-Tools Used => Manual
-Recommendation => It is recommended to be consistent and use named return values for all functions that return something unless it can be made explicitly clear why some have named return and others not 


#### NC-7 Constant or immutable variables should be capitalized

-Description => Some code instances capitalize variables like e.g named return variables

Capitalize royaltyRegistry
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L86

Capitalize stolenNftOracle
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L119

Capitalize factory 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L122

Capitalize royaltyRegistry
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L125

-Impact => This can lead to problems with code readability, consistency
-POC => Best Practise
-Tools Used => Manual
-Recommendation => It is recommended to capitalize immutable and constant variables 

#### NC-8 Spelling and grammar errors

-Description => There are instances of typos, misspelling, unclear grammar, mistakes etc in the code

exceute vs execute 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L169

correct to "will be used" this and other instances 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L67

amount not aount
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L251

Correct to "Sets whether"
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L573

-Impact => This can lead to problems with code readability, auditing, maintaning, or even errors if dev misinterprets comments
-POC => Best Practise
-Tools Used => Manual
-Recommendation => It is recommended to fix all typos, spelling and grammar errors ensure comments are well readable and easy to understand

 #### NC-9 Comment on all assembly 

-Description => Assembly is more complex and not all auditors, devs, code readers may understand it. It is important before every assembly block to comment what the code is doing. 

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L469

 #### NC-10 Function with irrelevant parameters 

-Description => Function which takes in arguments but not needed as its a simple getter function that returns a state variable  

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L750 


#### Low 1 Missing events for critical updates

The following onlyOwner state changes do not emit events
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141

-Impact => This can lead to lack of transparency, lack of monitoring, insufficient information o off chain tooling that may depend on emission of these critical events.
-POC => Best Practise
-Tools Used => Manual
-Recommendation => It is recommended to emit events for critical parameter updates. Fix for above instances or any other relevant that are useful to report changes to contract functionality

