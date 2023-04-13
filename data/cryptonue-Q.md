# [L] in Factory, create private pool doesn't check required variable already initialize in Constructor (or initialize function)

The following variable doesn't initialize up until the admin/owner manually set via dedicated function. This resulting when create private pool potential to failure since there is also no check in `create()` function if the variables already setted.

- PrivatePoolMetadata
- PrivatePoolImplementation

```js
File: Factory.sol
129:     function setPrivatePoolMetadata(address _privatePoolMetadata) public onlyOwner {
130:         privatePoolMetadata = _privatePoolMetadata;
131:     }
132:
133:     /// @notice Sets the private pool implementation contract that newly deployed proxies point to.
134:     /// @param _privatePoolImplementation The private pool implementation contract.
135:     function setPrivatePoolImplementation(address _privatePoolImplementation) public onlyOwner {
136:         privatePoolImplementation = _privatePoolImplementation;
137:     }
```


# [L] Solmate’s SafeTransferLib doesn’t check whether the ERC20 contract exists

Caviar use Solmate’s SafeTransferLib when transfering ERC20 token which contains a common issue.

Solmate’s SafeTransferLib, which is often used to interact with non-compliant/unsafe ERC20 tokens, does not check whether the ERC20 contract exists. The following code will not revert in case the token doesn’t exist (yet).

This is stated in the Solmate library:

https://github.com/transmissions11/solmate/blob/main/src/utils/SafeTransferLib.sol#L9

# [L] Two step transfer ownership

Caviar use solmate's Owned library (`import {Owned} from "solmate/auth/Owned.sol";`) which doesn't have a two step transfer ownership.

Recommend considering implementing a two step process where the owner or admin nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.


# [L] Unbounded protocol fee rate

the `setProtocolFeeRate` doesn't have a max value which can lead admin can increase the protocol fee to beyond normal

```js
File: Factory.sol
141:     function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
142:         protocolFeeRate = _protocolFeeRate;
143:     }
```

# [L] NatSpec incorrect

The NatSpec does not match what is being done by the function, missing @return for `protocolFeeAmount`

```js
File: PrivatePool.sol
252:     /// @return netInputAmount The amount of base tokens spent inclusive of fees.
253:     /// @return feeAmount The amount of base tokens spent on fees.
254:     function buy(
255:         uint256[] calldata tokenIds,
256:         uint256[] calldata tokenWeights,
257:         MerkleMultiProof calldata proof
258:     )
259:         public
260:         payable
261:         returns (
262:             uint256 netInputAmount,
263:             uint256 feeAmount,
264:             uint256 protocolFeeAmount
265:         )
266:     {
```
