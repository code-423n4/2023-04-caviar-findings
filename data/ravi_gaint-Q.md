# [L-01] Arbitrary fee recipient address can be set by anyone

### description:

The setFeeRecipient function in PrivatePool.sol allows anyone to set the fee recipient address, which is used to collect protocol fees. This can be exploited by an attacker to set their own address as the fee recipient and collect protocol fees that would otherwise be directed to the legitimate fee recipient.

### line link:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L107

### suggestion:

To prevent arbitrary address setting, it is recommended to add a permission check that only allows the owner or an authorized address to call the setFeeRecipient function. This can be done by adding a modifier that checks if the caller is the owner or authorized address, and applying this modifier to the setFeeRecipient function. Additionally, it is recommended to provide a way for the fee recipient address to be changed through a formalized process, such as a multi-signature scheme, to prevent unauthorized changes.





# [L-02] Lack of input validation in burn function



### description:

The burn function in the PrivatePool contract allows users to burn tokens and receive a proportionate share of the underlying tokens in the pool. However, this function does not validate whether the user has a sufficient balance of tokens to burn, which could potentially result in an underflow.

### line_link:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L87

### suggestion:

to add input validation to the burn function to ensure that the user has a sufficient balance of tokens to burn. This can be achieved by checking the user's balance against the amount of tokens they wish to burn and reverting the transaction if the balance is insufficient. Here's an example of how the function can be updated:

    function burn(uint256 _amount) external {
                require(_amount > 0, "Amount must be greater than zero");
                                        uint256 _balance = balances[msg.sender];
                                                                                require(_balance >= _amount, "Insufficient balance");
                                                                                                                            // Calculate the amount of underlying tokens the user is entitled to
                                                                                                                                                                                uint256 _underlyingAmount = _amount.mul(totalBalance()).div(totalSupply);
                                                                                                                                                                                                                                        // Update the user's balance and the total supply of tokens
                                                                                                                                                                                                                                                                                                        balances[msg.sender] = _balance.sub(_amount);
                                                                                                                                                                                                                                                                                                                                                                                    totalSupply = totalSupply.sub(_amount);
                                                                                                                                                                                                                                                                                                                                                                                                                                                                    // Transfer the underlying tokens to the user
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            underlying.safeTransfer(msg.sender, _underlyingAmount);
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        // Emit an event to indicate the successful burn
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            emit Burn(msg.sender, _amount, _underlyingAmount);
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        }







# [L-03] Lack of input validation in transferFrom function



### description:

The transferFrom function in PrivatePool.sol allows users to transfer tokens from the contract to a recipient without validating whether the recipient address is valid. This could potentially result in funds being transferred to an invalid address.

### line_link:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L191

### suggestion:

To prevent funds from being transferred to an invalid address, it is recommended to add input validation to the transferFrom function. This can be done by checking whether the recipient address is valid using the require statement. A sample implementation is provided below:

    function transferFrom(address from, address to, uint256 amount) public override {
                require(to != address(0), "Invalid address");
                                        super.transferFrom(from, to, amount);
                                                                            }
This will ensure that the transfer is only executed if the recipient address is valid.







# [L-04] Lack of allowance check in transferFrom function could allow token draining



### description:

The transferFrom function in PrivatePool.sol, lines 92-94, is used to transfer ERC20 tokens, but it does not check whether the caller has sufficient allowance to transfer the tokens. This could allow an attacker to drain the contract's tokens by calling the transferFrom function with a large amount, without having the required allowance.

### line_link:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L92-L94

### suggestion:

Add an allowance check before calling transferFrom to ensure that the caller has sufficient allowance to transfer the tokens. A possible implementation would be to use the allowance function of the ERC20 token and compare it to the amount being transferred eg:

    require(ERC20(token).allowance(msg.sender, address(this)) >= amount, "Insufficient allowance");
    This should be added before the transferFrom call,

    require(ERC20(token).allowance(msg.sender, address(this)) >= amount, "Insufficient allowance");
                require(ERC20(token).transferFrom(msg.sender, address(this), amount), "TransferFrom failed");
                Additionally, it is recommended to implement the ERC20 approve function to allow users to grant the contract a sufficient allowance before calling the transferFrom function.








# [L-05] Missing Input Token Approval Check in swap Function



### description:

The swap function in PrivatePool.sol allows users to swap one token for another, but it fails to check if the caller has approved the contract to spend the input token. This can lead to a potential vulnerability where an attacker can drain the contract's tokens by calling the `swap` function with a large amount.

### line_link:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L177-L181

### suggestion:

Before executing the token transfer, it is recommended to add a check that ensures the caller has approved the contract to spend the input token. This can be done by using the `require` statement to check if the contract has been approved to spend the required amount of input token. Here is an example of how this can be done:

    require(IERC20(inputToken).allowance(msg.sender, address(this)) >= inputAmount, "Input token allowance not sufficient");
        // Transfer the input token from the caller to the contract
            require(IERC20(inputToken).transferFrom(msg.sender, address(this), inputAmount), "Failed to transfer input token");
By adding this check, the `swap` function will only execute if the caller has approved the contract to spend the required amount of input token, thereby preventing any unauthorized transfer of tokens from the contract.






# [N-01] Lack of Input Validation in PrivatePool's Initialize Function

### description:

PrivatePool's initialize function does not perform input validation on the parameters tokenA and tokenB, which can lead to incorrect initialization of the pool.

### line link:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L49

### suggestion:

Add input validation to the initialize function to ensure that the parameters tokenA and tokenB are valid ERC20 tokens before initializing the pool. This can be achieved by using the require statement to check if the tokens are not zero addresses and implement ERC20 interface.

     function initialize(address _tokenA, address _tokenB, uint24 _fee, uint160 _sqrtPriceX96) public initializer {
                   require(_tokenA != address(0), "PrivatePool: tokenA cannot be zero address");
                                              require(_tokenB != address(0), "PrivatePool: tokenB cannot be zero address");
                                                                                          require(IERC20(_tokenA).totalSupply() > 0, "PrivatePool: tokenA has no supply");
                                                                                                                                                           require(IERC20(_tokenB).totalSupply() > 0, "PrivatePool: tokenB has no supply");
                                                                                                                                                                                                                                                     require(IERC20(_tokenA).decimals() == IERC20(_tokenB).decimals(), "PrivatePool: tokenA and tokenB decimals mismatch");
                                                                                                                                                                                                                                                                                                                                                                            // rest of the function
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                }





# [N-02] Potential front-running vulnerability in swap function



### description:

 The swap function in PrivatePool.sol uses a fixed price curve model to calculate the exchange rate and token amounts to be transferred between the sender and receiver. This calculation is based on the current reserve balances in the pool. An attacker could potentially exploit this by submitting a transaction with a higher gas price that executes the swap before the original transaction. This could cause the exchange rate to change, resulting in the original transaction receiving less tokens than expected.
 ### line_link:
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L90

### suggestion:

One way to mitigate this potential vulnerability is to add a timestamp parameter to the swap function, which would prevent any transaction from executing if the time difference between the current block timestamp and the provided timestamp is greater than a certain threshold. This would make it more difficult for an attacker to front-run a transaction by requiring them to predict the exact timestamp at which the original transaction will be executed. Another option would be to implement a different curve model that is less susceptible to front-running attacks, such as a constant sum curve. It is also important to note that gas price is not a reliable method for preventing front-running attacks, as gas prices can fluctuate rapidly and unpredictably.








# [N-03] Potential DoS vulnerability in getSpotPrice function due to zero reserve balance



### description:

The getSpotPrice function in PrivatePool.sol calculates the spot price for a given token pair using the reserve balances of the pair. However, if one of the reserve balances is set to zero, the function will always revert, which could be exploited to perform a denial of service (DoS) attack.

### line_link:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L103-L107

### suggestion:

To prevent a potential DoS attack, it's recommended to add a check to ensure that neither of the reserve balances are zero before executing the calculation. This can be achieved by adding a simple if statement before the calculation

    function getSpotPrice() external view returns (uint256) {
                uint256 _reserve0 = reserve0;
                                        uint256 _reserve1 = reserve1;
                                                                                require(_reserve0 != 0 && _reserve1 != 0, "Reserve balances cannot be zero");
                                                                                                                                            return _reserve1.mul(PRECISION_FACTOR).div(_reserve0);
                                                                                                                                                                                                                            }
                                                                                                                                                                                                                            With this check, the function will revert with an error message if either of the reserve balances is zero, preventing a potential DoS attack.







# [N-04] Potential Reentrancy Vulnerability in withdrawFees Function



### description:

The withdrawFees function in the PrivatePool contract withdraws accumulated fees from the contract and transfers them to the fee receiver. The function uses the transfer method to transfer the fees to the receiver, which could result in a reentrancy vulnerability.

### line_link:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L310

### suggestion:

to use send or call instead of transfer to transfer the funds in order to avoid potential reentrancy attacks. Additionally, it's important to ensure that any state changes are made before transferring the funds to prevent any unwanted side effects.






# [N-05] Missing Event Emission for Protocol Fees in Swap Function



### description:

The Swap function in the PrivatePool contract emits a Swap event with the input and output tokens and the trade amounts, but it does not emit an event for the protocol fee charged by the contract. This could make it difficult to audit the fees collected by the contract.

### line_link:

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L186-L188

### suggestion:

It is recommended to add an additional event emission for the protocol fee charged by the contract. This will make it easier for auditors and users to track the fees collected by the contract. The event should include the token address and the amount of the fee charged.