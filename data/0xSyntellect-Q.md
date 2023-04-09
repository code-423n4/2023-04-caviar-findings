In PrivatePool contract, when calling the public deposit() function with baseToken=ETH, the first check requires that the caller should pass the same value of ETH sent to the second argument of the function: baseTokenAmount.



    if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken !=address(0))) {
                revert InvalidEthAmount();
            }
This is redundant as it only checks if the arbitrary amount of baseTokenAmount is equal to the value of ETH sent. There are 2 issues with this:

1)It doesn't provide protection against wrong ETH input amounts. 

2)Caller can send the correct amount of ETH but in case they fail to input the same value as baseTokenAmount this check would revert causing unnecessary failure of execution
