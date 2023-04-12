Findings: Incorrect fee condition. PrivatePool feeRate can be set to 50% on `setFeeRate` and `initialize`

During Private Pool initialize it's assumed "check that the fee rate is less than 50%" in comments. However, fee can be set at 50% 
  
Code: https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L172-L173

- POC

forge test --match-contract SettersTest --match-test test_setFeeRate_setsFeeRate_5000

``` solidity

       function test_setFeeRate_setsFeeRate_5000() public {
        // arrange
        uint16 feeRate = 5_000;

        // act
        privatePool.setFeeRate(feeRate);

        // assert - should expect aa pass even though commented "check that the fee rate is less than 50%"
        assertEq(privatePool.feeRate(), feeRate, "Should have set fee rate");
    }


```

Recommendation: 
- `if (_feeRate >= 5_000) revert FeeRateTooHigh();`

-----

## Findings:
### changeFeeQuote assumes all base tokens will be of decimals of at > 4 decimals. However this would lead to issues whereby baseToken is of 2 decimal place.

code: https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L733-L734

---
### findings: Buy() - Buying NFT from the pool will become locked (always revert) if protocol amount is too high, or users can loose unexpected funds.

This is as a result of the setProtocolFeeRate exploit whereby
factory.setProtocolFeeRate(type(uint16).max),
given there is no cap on the fee rate.

---

Other Findings:

### No real reference in the doc that fee rate must be below 50% only in the code.
- Code: https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L171-L173
  
- `protocolFeeAmount` is not referenced in the buy() natspec comments

----
### A lot of missing emit events within critical functions
----
### Cache storage variables
  - baseToken used instead store value in cache to reduce gas requred to read a variable from storage uint256 _baseToken = baseToken.
  - code: https://github.com/code-423n4/2023-04-caviar/blob/dc3c76674133c0c5f50348a3862431292c387654/src/PrivatePool.sol#L648-L649

---
### Floating point pragma version contracts

---