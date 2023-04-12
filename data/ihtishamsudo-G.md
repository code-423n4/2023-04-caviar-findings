
# [G-01]

## Use double IF Statements instead of && 

if the IF Statement has a logical AND (&&) Operation but is not followed by the else statement, it can be replaced with 2 if statements to reduce gas cost

###### Before

```
if (block.timestamp > deadline && deadline != 0) 
{
    revert DeadlinePassed();
}
```

**It consumes more gas than**

###### After

```
if (block.timestamp > deadline) {
    if (deadline != 0) {
        revert DeadlinePassed();
    }
}
```

##### References

[EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L101)
[EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L154)
[EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L228)
[EthRouter.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/EthRouter.sol#L256)

# [G-02]

## x += y costs more gas than x = x + y & same for x -= y

##### References

[PrivatePool.sol#L230](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L230)
[PrivatePool.sol#L231](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L231)
[PrivatePool.sol#L247](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L247)
[PrivatePool.sol#L252](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L252)
[PrivatePool.sol#L324](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L324)
[PrivatePool.sol#L341](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L341)
[PrivatePool.sol#L678](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L678)




