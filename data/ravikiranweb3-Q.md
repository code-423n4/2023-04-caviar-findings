a) IStolenNftOracle Implementation and storing reference in Private Pool contract
================================================================================

As an example, look at the demo implementation of IStolenNftOracle in the test folder, where setStolenNft() can be called by any one and flag all the tokens in the as stolen and making the StolenNftOracle as unusable.

Look at the below code as example:
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L118#L119

Ofcouse the Private Pool has the ability to disable the calls to IStolenNftOracle and bypass the validation logic for stolen. Since the oracle is set at private pool level, the logic could also be customised for pool requirements.

The problem is, currently the "stolenNftOracle" is declared as immutable. 
https://github.com/code-423n4/2023-04-caviar/blob/main/test/shared/StolenNftOracle.sol#L11#L13

Suggestion:
==========
Suggestion is to instead make it mutable and let the owner of the private pool manage it.

Incase some thing is wrong with the oracle, it gives owner the flexibility to disable it immediately and later deploy a new implementation and link it to the pool for flagging stolen tokens.





