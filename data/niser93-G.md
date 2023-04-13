## GAS-01 - Use equivalent expression

### Founded in [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)
* [[L101-L103]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101-L103)
* [[L154-L156]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154-L156)
* [[L228-L230]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228-L230)
* [[L256-L258]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256-L258)
```
if (block.timestamp > deadline && deadline != 0) {
	revert DeadlinePassed();
}
```


### Description
```
if (block.timestamp > deadline && deadline != 0) {
        revert DeadlinePassed();
        }
```
becomes
```
if (!(block.timestamp <= deadline || deadline == 0)) {
            revert DeadlinePassed();
        }
```

### Proof of concept
Expression equality (using boolean formulas)
```
A>B && C!=0           is equivalent to

!(!(A>B && C!=0))           is equivalent to

!(!(A>B) || !(C!=0))           is equivalent to

!(A<=B || C==0)
```

With Remix IDE and compiling following code using optimization with 10000:

```
contract Attempt{
    error DeadlinePassed();

    function tryGas(uint256 deadline) public {
        if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
    }
}

Execution cost: 290
```

```
contract Attempt{
    error DeadlinePassed();

    function tryGas(uint256 deadline) public {
        if (!(block.timestamp <= deadline || deadline == 0)) {
            revert DeadlinePassed();
        }
    }
}
Execution cost: 284
```


## GAS-02 - Require costs less gas than revert + Error

### Founded in [EthRouter.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol)
* [[L101-L103]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101-L103)
* [[L154-L156]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154-L156)
* [[L228-L230]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228-L230)
* [[L256-L258]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256-L258)
```
if (block.timestamp > deadline && deadline != 0) {
	revert DeadlinePassed();
}
```

* [[L203-L105]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L203-L205)
```
if (address(this).balance < minOutputAmount) {
	revert OutputAmountTooSmall();
}
```

* [[L234-L236]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L234-L236)
```
if (price > maxPrice || price < minPrice) {
	revert PriceOutOfRange();
}
```
    
* [[L314]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L314)
```
if (royaltyFee > salePrice) revert  InvalidRoyaltyFee();
```

### Founded in [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

* [[L128-L130]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L128-L130)
```
if (msg.sender != Factory(factory).ownerOf(uint160(address(this)))) {
	revert Unauthorized();
}
```

* [[L169]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L169)
```
if (initialized) revert  AlreadyInitialized();
```

* [[L172]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L172)
```
if (_feeRate >  5_000) revert  FeeRateTooHigh();
```

* [[L225]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L225)
```
if (baseToken !=  address(0) &&  msg.value  >  0) revert  InvalidEthAmount();
```

* [[L262]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L262)
```
if (msg.value  < netInputAmount) revert  InvalidEthAmount();
```

* [[L397]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L397)
```
if (baseToken !=  address(0) &&  msg.value  >  0) revert  InvalidEthAmount();
```

* [[L413]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L413)
```
if (inputWeightSum < outputWeightSum) revert  InsufficientInputWeight();
```

* [[L429]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L429)
```
if (msg.value  < feeAmount + protocolFeeAmount) revert  InvalidEthAmount();
```

* [[L489-L491]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L489-L491)
```
if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {
	revert InvalidEthAmount();
}
```

* [[L564]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L564)
```
if (newFeeRate >  5_000) revert  FeeRateTooHigh();
```

* [[L629]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L629)
```
if (!availableForFlashLoan(token, tokenId)) revert  NotAvailableForFlashLoan();
```

* [[L635]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L635)
```
if (baseToken ==  address(0) &&  msg.value  < fee) revert  InvalidEthAmount();
```

* [[L645]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L645)
```
if (!success) revert  FlashLoanFailed();
```

* [[L682-L684]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L683-L684)
```
if (!MerkleProofLib.verifyMultiProof(proof.proof, merkleRoot, leafs, proof.flags)) {
	revert InvalidMerkleProof();
}
```

* [[L791]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L791)
```
if (royaltyFee > salePrice) revert  InvalidRoyaltyFee();
```



### Description
Using require instead of revert error (as used, they are equivalent, see also https://stackoverflow.com/questions/72027635/ethereum-solidity-whats-the-difference-between-require-and-revert-error)

For example, [EthRouter.sol#L101-L103](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101-L103)

```
if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
```

becames

```
require(!(block.timestamp > deadline && deadline != 0))
```

or better (see GAS-01 report)

```
require(block.timestamp <= deadline || deadline == 0))
```

### Proof of concept

With Remix IDE and compiling following code using optimization with 10000:

```
contract Attempt{
    error DeadlinePassed();

    function tryGas(uint256 deadline) public {
        if (block.timestamp > deadline && deadline != 0) {
            revert DeadlinePassed();
        }
    }
}

Execution cost: 290
```

```
contract Attempt{
    function tryGas(uint256 deadline) public {
        require(!(block.timestamp > deadline && deadline != 0));
    }
}

Execution cost: 251
```



## GAS-03 - Avoid to use != and use == instead

### Founded in [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)
* [[L272-284]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L272-L284)
```
for (uint256 i = 0; i < tokenIds.length; i++) {
	// get the royalty fee for the NFT
	(uint256 royaltyFee, address recipient) = _getRoyalty(tokenIds[i], salePrice);
	// transfer the royalty fee to the recipient if it's greater than 0
	if (royaltyFee > 0 && recipient != address(0)) {
		if (baseToken != address(0)) {
			ERC20(baseToken).safeTransfer(recipient, royaltyFee);
		} else {
			recipient.safeTransferETH(royaltyFee);
		}
	}
}
```

* [[L254-269]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L254-L269)
```
if (baseToken != address(0)) {
	// transfer the base token from the caller to the contract
	ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount);
	
	// if the protocol fee is set then pay the protocol fee
	if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
} else {
	// check that the caller sent enough ETH to cover the net required input
	if (msg.value < netInputAmount) revert InvalidEthAmount();
	
	// if the protocol fee is set then pay the protocol fee
	if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
	
	// refund any excess ETH to the caller
	if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);
}
```

* [[L345-349]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L345-L349)
```
if (baseToken != address(0)) {
	ERC20(baseToken).safeTransfer(recipient, royaltyFee);
} else {
	recipient.safeTransferETH(royaltyFee);
}
```

* [[L421-438]](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L421-L438)
```
if (baseToken != address(0)) {
	// transfer the fee amount of base tokens from the caller
	ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);
	
	// if the protocol fee is non-zero then transfer the protocol fee to the factory
	if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);
	} else {
	// check that the caller sent enough ETH to cover the fee amount and protocol fee
	if (msg.value < feeAmount + protocolFeeAmount) revert InvalidEthAmount();
	
	// if the protocol fee is non-zero then transfer the protocol fee to the factory
	if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
	
	// refund any excess ETH to the caller
	if (msg.value > feeAmount + protocolFeeAmount) {
		msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);
	}
}
```

### Description
Using == instead of != saves 2 gas cost

```
if (baseToken != address(0)) {
	ERC20(baseToken).safeTransfer(recipient, royaltyFee);
} else {
	recipient.safeTransferETH(royaltyFee);
}
```

becames

```
if (baseToken == address(0)) {
	recipient.safeTransferETH(royaltyFee);
} else {
	ERC20(baseToken).safeTransfer(recipient, royaltyFee);
}
```

### Proof of concept

With Remix IDE and compiling following code using optimization with 10000:

```
contract Attempt{
	error Error1();
	error Error2();
	function tryGas(uint256 royaltyFee, address recipient, address baseToken) public {
		if (royaltyFee > 0 && recipient != address(0)) {
			if (baseToken != address(0)) {
				revert Error1();
			} else {
				revert Error2();
			}
		}
	}
}

Execution cost: 501
```

```
contract Attempt{
	error Error1();
	error Error2();
	function tryGas(uint256 royaltyFee, address recipient, address baseToken) public {
		if (royaltyFee > 0 && recipient != address(0)) {
			if (baseToken == address(0)) {
				revert Error2();
			} else {
				revert Error1();
			}
		}
	}
}

Execution cost: 499
```

In this way, contract saves NOT operation, i.e. 2 gas, for each loop


## GAS-04 - Incomplete check of msg.value with respect fees

### Founded in [PrivatePool.sol](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol)

* [L421-L438](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L421-L438)


```
if (baseToken != address(0)) {
	// transfer the fee amount of base tokens from the caller
	ERC20(baseToken).safeTransferFrom(msg.sender, address(this), feeAmount);
	
	// if the protocol fee is non-zero then transfer the protocol fee to the factory
	if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);
	} else {
	// check that the caller sent enough ETH to cover the fee amount and protocol fee
	if (msg.value < feeAmount + protocolFeeAmount) revert InvalidEthAmount();
	
	// if the protocol fee is non-zero then transfer the protocol fee to the factory
	if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
	
	// refund any excess ETH to the caller
	if (msg.value > feeAmount + protocolFeeAmount) {
	msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);
	}
}
```

 ### Description

```
if (msg.value > feeAmount + protocolFeeAmount)
```
is useless after follow check
```
if (msg.value < feeAmount + protocolFeeAmount) revert InvalidEthAmount();
```

The only case in which both are false is for
```
msg.value == feeAmount + protocolFeeAmount    
```
but in this case
```
msg.sender.safeTransferETH(msg.value - feeAmount - protocolFeeAmount);
```
becames

```
msg.sender.safeTransferETH(0);
```


### Proof of concept

With Remix IDE and compiling following code using optimization with 10000:

```
contract Attempt{
	error Error1();
	error safeTransferETH();

	function tryGas(uint256 feeAmount,  uint256 protocolFeeAmount,  uint256 msg_value,  address baseToken)  public  {
		if  (baseToken !=  address(0))  {
			if  (msg_value < feeAmount + protocolFeeAmount)  revert Error1();
			if  (msg_value > feeAmount + protocolFeeAmount)  {
				revert safeTransferETH();
			}
		}
	}
}

Execution cost: 561
```

```
contract Attempt{
	error Error1();
	error safeTransferETH();

	function tryGas(uint256 feeAmount,  uint256 protocolFeeAmount,  uint256 msg_value,  address baseToken)  public  {
		if  (baseToken !=  address(0))  {
			if  (msg_value < feeAmount + protocolFeeAmount)  revert Error1();
			revert safeTransferETH();
		}
	}
}

Execution cost: 464
```