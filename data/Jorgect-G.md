# GAS REPORT

## G-1 Use internal functions insted of public function.
 
internal functions are likely to be less gas expensive than public functions. This is because internal functions can only be called from within the same contract and do not need to go through the EVM's message-passing system, which can add additional gas costs.

it should change the functions below not only because internal function are more cheaper, also for security reason:


```
contract Factory.sol line 136
function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }
```

```
contract PrivateFacotry.sol line 877
 function sumWeightsAndValidateProof(
        uint256[] memory tokenIds,
        uint256[] memory tokenWeights,
        MerkleMultiProof memory proof
    ) public view returns (uint256) 
```

```
contract PrivateFacotry.sol line 921
  function buyQuote(
        uint256 outputAmount
    )
        public
        view
        returns (
            uint256 netInputAmount,
            uint256 feeAmount,
            uint256 protocolFeeAmount
        )
```

```
contract PrivateFacotry.sol line 978
  function changeFeeQuote(
        uint256 inputAmount
    ) public view returns (uint256 feeAmount, uint256 protocolFeeAmount)
```

### the next functions can be change to external
1. contract PrivateFacotry.sol line 254 
2. contract PrivateFacotry.sol line 507
3. contract PrivateFacotry.sol line 385
4. contract PrivateFacotry.sol line 655
5. contract PrivateFacotry.sol line 699
6. contract PrivateFacotry.sol line 734
7. contract PrivateFacotry.sol line 752
8. contract PrivateFacotry.sol line 764
9. contract PrivateFacotry.sol line 778
10. contract PrivateFacotry.sol line 791
11. contract PrivateFacotry.sol line 807