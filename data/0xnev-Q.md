### Issues Template
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Code changes |
| O | Ordinary | Commonly found issues |

| Total Found Issues | 5 |
|:--:|:--:|

### Low Risk Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [L-01] | Remove unecessary `receive() payable` function in EthRouter | 1 |
| [L-02] | Solmate’s `SafeTransferLib `doesn’t check whether the ERC20 contract exists| 21 |
| [L-03] | Confusing deadline comments| 4 |

| Total Low Risk Issues | 3 |
|:--:|:--:|

### Non-Critical Template
| Count | Title | Instances |
|:--:|:-------|:--:|
| [N-01] | Missing event for critical parameters init and change | 3 |
| [N-02] | Add timelock to critical functions | 3 |

| Total Non-Critical Issues | 2 |
|:--:|:--:|

### [L-01] Remove unecessary `receive() payable` function in EthRouter.sol
[EthRouter.sol#L88](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88)
```solidity
/EthRouter.sol
88:    receive() external payable {}
```
There is no means of withdrawing ether sent to the `EthRouter.sol` contract in the event where ether is accidentally sent to it. This can lead to accidental locking of funds. Suggest removing unecessary `receive() external payable{}` function or implement withdrawal mechanism. 
 
### [L-02] Solmate’s `SafeTransferLib `doesn’t check whether the ERC20 contract exists
[Factory.sol#L120](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L120)
```solidity
21 results - 3 files

/Factory.sol
120:            ERC721(_nft).safeTransferFrom(msg.sender, address(privatePool), tokenIds[i]);
```
[EthRouter.sol#L136](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L136)
[EthRouter.sol#L162](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L162)
[EthRouter.sol#L266](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L266)
[EthRouter.sol#L285](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L285)
```solidity
/EthRouter.sol
136:                ERC721(buys[i].nft).safeTransferFrom(address(this), msg.sender, buys[i].tokenIds[j]);

162:                ERC721(sells[i].nft).safeTransferFrom(msg.sender, address(this), sells[i].tokenIds[j]);

266:                ERC721(_change.nft).safeTransferFrom(msg.sender, address(this), _change.inputTokenIds[j]);

285:                ERC721(_change.nft).safeTransferFrom(address(this), msg.sender, _change.outputTokenIds[j]);
```
[PrivatePool.sol#L240](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L240)
[PrivatePool.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L256)
[PrivatePool.sol#L259](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L259)
[PrivatePool.sol#L279](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L279)
[PrivatePool.sol#L331](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L331)
[PrivatePool.sol#L346](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L346)
[PrivatePool.sol#L368](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L368)
[PrivatePool.sol#L423](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L423)
[PrivatePool.sol#L426](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L426)
[PrivatePool.sol#L442](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L442)
[PrivatePool.sol#L497](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L497)
[PrivatePool.sol#L502](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L502)
[PrivatePool.sol#L519](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L519)
[PrivatePool.sol#L638](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L638)
[PrivatePool.sol#L648](https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L648)
```solidity
/PrivatePool.sol
240:            ERC721(nft).safeTransferFrom

256:            ERC20(baseToken).safeTransferFrom

259:                ERC20(baseToken).safeTransfer

279:                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);

331:            ERC721(nft).safeTransferFrom

346:                        ERC20(baseToken).safeTransfer(recipient, royaltyFee);

368:                ERC20(baseToken).safeTransfer

423:            ERC20(baseToken).safeTransferFrom

426:                ERC20(baseToken).safeTransferFrom

442:            ERC721(nft).safeTransferFrom

447:            ERC721(nft).safeTransferFrom

497:            ERC721(nft).safeTransferFrom

502:            ERC20(baseToken).safeTransferFrom

519:            ERC721(_nft).safeTransferFrom

638:        ERC721(token).safeTransferFrom

648:        ERC721(token).safeTransferFrom
```
Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The above instances will not revert in case the token doesn’t exist (yet).

Consider adding a check for contract existence.


### [L-03] Confusing deadline comments
[EthRouter.sol#L101](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L101)
[EthRouter.sol#L154](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L154)
[EthRouter.sol#L228](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L228)
[EthRouter.sol#L256](https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L256)
```solidity
4 results - 1 file

/EthRouter.sol
101:        if (block.timestamp > deadline && deadline != 0)

154:        if (block.timestamp > deadline && deadline != 0)

228:        if (block.timestamp > deadline && deadline != 0)

256:        if (block.timestamp > deadline && deadline != 0)
```
Many of the core contract functions of `EthRouter.sol` implement a deadline for the transaction to be mined and will revert if `block.timestamp > deadline`. In the comments, protocol states to check deadline has pass (if any) indicating the possibility of allowing users to not set a `deadline`, where `deadline = 0` or  `block.timestamp > deadline`. However, in the function comments, it says deadline set to 0 is to be ignored. In this case, since `block.timestamp > deadline` will always pass when `deadline = 0`, it does not matter if protocol implements the check for `deadline != 0` and hence it is redundant.

### [N-01] Missing event for critical parameters init and change
[Factory.sol#L129-L143](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129-L143)
```solidity
3 results - 1 file

/Factory.sol
129:    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
130:        privatePoolMetadata = _privatePoolMetadata;
131:    }

135:    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
136:        privatePoolImplementation = _privatePoolImplementation;
137:    }

141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
142:        protocolFeeRate = _protocolFeeRate;
143:    }
```

Events help non-contract tools to track changes, and events prevent users from being surprised by changes

Consider adding Event-Emit

### [N-02] Add timelock to critical functions
[Factory.sol#L129-L143](https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L129-L143)
```solidity
3 results - 1 file

/Factory.sol
129:    function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
130:        privatePoolMetadata = _privatePoolMetadata;
131:    }

135:    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
136:        privatePoolImplementation = _privatePoolImplementation;
137:    }

141:    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
142:        protocolFeeRate = _protocolFeeRate;
143:    }
```

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).


