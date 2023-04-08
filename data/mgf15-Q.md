
## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Constants should be defined rather than using magic numbers | 1 |
### [NC-1] Constants should be defined rather than using magic numbers

*Instances (1)*:
```solidity
File: 2023-04-caviar/src/PrivatePool.sol

744:         uint256 exponent = baseToken == address(0) ? 18 : (36 - ERC20(baseToken).decimals());

```


## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1) | Initializers could be front-run | 2 |
| [L-2](#L-2) | Unsafe ERC20 operation(s) | 5 |

### [L-1] Initializers could be front-run
Initializers could be front-run, allowing an attacker to either set their own values, take ownership of the contract, and in the best case forcing a re-deployment

*Instances (2)*:
```solidity
File: 2023-04-caviar/src/Factory.sol

98:         privatePool.initialize(

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

157:     function initialize(

```

### [L-2] Unsafe ERC20 operation(s)

*Instances (5)*:
```solidity
File: 2023-04-caviar/src/Factory.sol

115:             ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);

152:             ERC20(token).transfer(msg.sender, amount);

```

```solidity
File: 2023-04-caviar/src/PrivatePool.sol

365:             ERC20(baseToken).transfer(msg.sender, netOutputAmount);

527:             ERC20(token).transfer(msg.sender, tokenAmount);

651:         if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

```
