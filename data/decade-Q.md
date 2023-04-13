## Q/A 
---
#### 1. ProtocolFee should have max limit check before being initialized or updated.
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L141
```
    /// @notice Sets the protocol fee that is taken on each buy/sell/change. It's in basis points: 350 = 3.5%.
    /// @param _protocolFeeRate The protocol fee.
    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }
```
-- ProtocolFee set by admin should have a check for max limit. 
-- Admin should not be able to set protocol fees above a certain limit. This allows protection against future admin key theft. 

---

#### 2. Place 2-step verification before changing private pool implementation.
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135 
```
    function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
        privatePoolImplementation = _privatePoolImplementation;
    }
```
-- As theft of admin key could lead to Parity bug type situation where faulty implementation will affect **all the future pools**.
https://github.com/openethereum/parity-ethereum/issues/6995

---

#### 3. Place 2-step ownership transfer for factory contract .
-- Factory contract currently uses direct ownership transfer via [solmate](https://github.com/transmissions11/solmate/blob/1b3adf677e7e383cc684b5d5bd441da86bf4bf1c/src/auth/Owned.sol), which directly transfers ownership to the given address. It is recommended not to use this approach as direct transfer of ownership that could lead to misconfiguration.
-- 2 step ownership transfer supports protocol not lose ownership to a dead address.

---

#### 4. No check present for length mismatch of dynamic arrays.
-- Where - [`sumWeightsAndValidateProof()`](https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L661-L687)
-- No check is placed for `uint256[] memory tokenIds` and `uint256[] memory tokenWeights` equal lengths. 
-- This could lead unexpected behavior especially users not using the protocol's frontend. 
-- Ideally, `tokenIds.length==tokenWeights.length` check should be there, to avoid confusion. 

---

#### 5. ChangeFee should have max limit check before being initialized.
-- ChangeFee initialized in private pool by owner should also have max limit check. 
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/PrivatePool.sol#L179
```
        changeFee = _changeFee;
```

Instead, similiar check should be there.
```
        // check that the fee rate is less than 50%
        if (_feeRate > 5_000) revert FeeRateTooHigh();
```
-- Malicious user shouldnot be able to charge extremely high changeFee. 

---

#### 6. Irregularities in implementation of different private pools. 
-- https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L92
 As the above line suggests, all private pools are deployed with current implementation. While older private pools will still have older implementation of private pools. 
-- This could result in confusion and team would require to keep track of pools with different implementation. 
-- Keeping track of privatepools and their respective implementation in the factory contract could also provide better user experience without relying on 3rd party. 

---

#### 7. No events are indexed. 
-- Certain important events such as Buy, Sell, Change should have atleast one indexed arguments so that it becomes easier for protocol during data analysis. 

