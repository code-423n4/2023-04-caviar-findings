
## Summary

## Non Critical Issues


|  | Issue | Instances |
| --- | --- | ---|
| 1 | Functions not called internally should be marked `external` | 6 |
| 2 | Return value from `transfer` is ignored | 3 |
| 3 | Input Validation not done | 2 |

## Non Critical Issues

# [NC-01] Functions not called internally should be marked `external`

*There are 6 instances of this issue:*

```javascript
File: src/Factory.sol

129    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner

135    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner 

141    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner
        
148    function withdraw(address token, uint256 amount) public onlyOwner

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L129

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L148

```javascript
File: src/PrivatePool.sol

514    function withdraw(address _nft, uint256[] calldata tokenIds, address token, uint256 tokenAmount) public onlyOwner

742    function price() public view returns (uint256)

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L514

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L742

# [NC-02] Return value from `transfer` is ignored

Not all IERC20 implementations revert() when there’s a failure in the `transfer()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything.

*There are 3 instances of this issue:*

```javascript
File: src/Factory.sol

152            ERC20(token).transfer(msg.sender, amount);
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L152

```javascript
File: src/PrivatePool.sol

365            ERC20(baseToken).transfer(msg.sender, netOutputAmount);

527            ERC20(token).transfer(msg.sender, tokenAmount);

```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L365

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L527


# [NC-03] Input Validation not done

Input should be properly validated to ensure correct behaviour. No check for `amount > 0`.

*There is 2 instance of this issue:*

```javascript
File: src/Factory.sol

150            msg.sender.safeTransferETH(amount);

152            ERC20(token).transfer(msg.sender, amount);
```

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L150

https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L152
