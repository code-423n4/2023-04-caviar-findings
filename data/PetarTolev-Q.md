## Low Risk Findings Overview
| ID | Finding | Instances |
| -- | ------- | --------- |
| [L-01] | Missing checks for `address(0x0)` when assigning values to `address` state variables | 3 |

*Total: 3 instances over 1 issues*

## Non-critical Findings Overview
| ID | Finding | Instances |
| -- | ------- | --------- |
| [N-01] | Unused input parameters | 1 |
| [N-02] | NatSpec is incomplete | 8 |

*Total: 9 instances over 2 issues*

---

# Low Risk Findings

## [L-01] Missing checks for `address(0x0)` when assigning values to `address` state variables

Checking addresses against the zero address during initialization or when setting them is a security best practice. However, such checks are missing in the initialization or modification of address variables in many places.

The impact of allowing zero addresses can lead to contract reverts and force re-deployments if there are no setters for such address variables.

*There are 3 instances of this issue:* \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L130 \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L136 \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L159 

# Non-critical Findings

### [N-01] Unused input parameters
*There are 3 instances of this issue:* \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L750

### [N-02] NatSpec is incomplete

Some parts of the NatSpec documentation have missing input parameters or output descriptions. When documenting a function, make sure to provide clear and concise descriptions of the input parameters and expected output.

*There are 8 instances of this issue:* \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L157 \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L306 \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L393 \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L697 \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L17 \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L35 \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L55 \
https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112 
