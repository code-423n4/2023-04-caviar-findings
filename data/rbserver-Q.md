## QA REPORT

| |Issue|
|-|:-|
| [01] | FEE-ON-TRANSFER TOKENS ARE NOT SUPPORTED |
| [02] | `PrivatePool.flashLoan` FUNCTION DOES NOT CHECK `baseToken != address(0) && msg.value > 0` |
| [03] | `PrivatePool.flashLoan` FUNCTION DOES NOT REFUND UNUSED ETH AMOUNTS TO USERS |
| [04] | `Factory.setProtocolFeeRate` FUNCTION DOES NOT HAVE A LIMIT FOR SETTING `protocolFeeRate` |
| [05] | ETH AMOUNTS CAN BE SENT TO `EthRouter` AND `Factory` CONTRACTS ACCIDENTALLY |
| [06] | MISSING `address(0)` CHECKS FOR CRITICAL ADDRESS INPUTS |
| [07] | CASTING `factory` TO `address` IS REDUNDANT |
| [08] | CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS |
| [09] | FLOATING PRAGMAS |
| [10] | INCOMPLETE NATSPEC COMMENTS |
| [11] | MISSING NATSPEC COMMENTS |

## [01] FEE-ON-TRANSFER TOKENS ARE NOT SUPPORTED
For a fee-on-transfer token, calling the following `PrivatePool.buy` function can transfer an token amount that is less than `netInputAmount` to the `PrivatePool` contract because the transfer fee is deducted after `ERC20(baseToken).safeTransferFrom(msg.sender, address(this), netInputAmount)` is executed. However, `virtualBaseTokenReserves` is still increased by `uint128(netInputAmount - feeAmount - protocolFeeAmount)`. The resulting `virtualBaseTokenReserves` will be larger than the actual token amount owned by the `PrivatePool` contract after transferring out `feeAmount` and `protocolFeeAmount` tokens. This will lead to incorrect calculations when calling functions like `PrivatePool.buyQuote` and `PrivatePool.sellQuote` that depend on `virtualBaseTokenReserves`. For example, when calling the `PrivatePool.buy` function, which calls `PrivatePool.buyQuote` function, again would calculate an incorrect `netInputAmount` for the user to pay since `virtualBaseTokenReserves` is already higher than it should be; as a result, the user can pay too much.

As a mitigation, the `PrivatePool.buy` function can be updated to record the `PrivatePool` contract's balance of such fee-on-transfer token before and after the token transfer, and the token balance difference can then be used to update `virtualBaseTokenReserves`.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L289
```solidity
    function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
    {
        // ~~~ Checks ~~~ //

        // calculate the sum of weights of the NFTs to buy
        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

        // calculate the required net input amount and fee amount
        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);
        ...
        // ~~~ Effects ~~~ //

        // update the virtual reserves
        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount);
        virtualNftReserves -= uint128(weightSum);

        // ~~~ Interactions ~~~ //

        // calculate the sale price (assume it's the same for each NFT even if weights differ)
        uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
        ...
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
        ...
    }
```

## [02] `PrivatePool.flashLoan` FUNCTION DOES NOT CHECK `baseToken != address(0) && msg.value > 0`
The following `PrivatePool.flashLoan` function executes `if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount()` so it is possible that the user sends some ETH while `baseToken` is an ERC20 token. This is different than functions like `PrivatePool.change` below, which executes `if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount()`. If the user sends some ETH while `baseToken` is an ERC20 token when calling the `PrivatePool.flashLoan` function, the user loses such sent ETH amount.

As a mitigation, the `PrivatePool.flashLoan` function can be updated to additionally check if `baseToken != address(0) && msg.value > 0` is true. If so, calling this function should revert.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L623-L654
```solidity
    function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data)
        external
        payable
        returns (bool)
    {
        // check that the NFT is available for a flash loan
        if (!availableForFlashLoan(token, tokenId)) revert NotAvailableForFlashLoan();

        // calculate the fee
        uint256 fee = flashFee(token, tokenId);

        // if base token is ETH then check that caller sent enough for the fee
        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();
        ...
    }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L385-L452
```solidity
    function change(
        uint256[] memory inputTokenIds,
        uint256[] memory inputTokenWeights,
        MerkleMultiProof memory inputProof,
        IStolenNftOracle.Message[] memory stolenNftProofs,
        uint256[] memory outputTokenIds,
        uint256[] memory outputTokenWeights,
        MerkleMultiProof memory outputProof
    ) public payable returns (uint256 feeAmount, uint256 protocolFeeAmount) {
        ...
        // check that the caller sent 0 ETH if base token is not ETH
        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
        ...
    }
```

## [03] `PrivatePool.flashLoan` FUNCTION DOES NOT REFUND UNUSED ETH AMOUNTS TO USERS
The following `PrivatePool.flashLoan` function does not refund the sent extra ETH amount that is unused, which is unlike functions like `PrivatePool.buy` below, which does refund the user in this situation. Users, who are familiar with the refund mechanism provided by functions like `PrivatePool.buy`, can send more than enough ETH when calling the `PrivatePool.flashLoan` function. However, since these unused ETH amounts sent by the users will not be refunded to them, these users lose such unused ETH amounts.

As a mitigation, the `PrivatePool.flashLoan` function can be updated to refund the unused ETH amounts to users.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L623-L654
```solidity
    function flashLoan(IERC3156FlashBorrower receiver, address token, uint256 tokenId, bytes calldata data)
        external
        payable
        returns (bool)
    {
        // check that the NFT is available for a flash loan
        if (!availableForFlashLoan(token, tokenId)) revert NotAvailableForFlashLoan();

        // calculate the fee
        uint256 fee = flashFee(token, tokenId);

        // if base token is ETH then check that caller sent enough for the fee
        if (baseToken == address(0) && msg.value < fee) revert InvalidEthAmount();

        // transfer the NFT to the borrower
        ERC721(token).safeTransferFrom(address(this), address(receiver), tokenId);

        // call the borrower
        bool success =
            receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");

        // check that flashloan was successful
        if (!success) revert FlashLoanFailed();

        // transfer the NFT from the borrower
        ERC721(token).safeTransferFrom(address(receiver), address(this), tokenId);

        // transfer the fee from the borrower
        if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

        return success;
    }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L211-L289
```solidity
    function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
    {
        ...
        if (baseToken != address(0)) {
            ...
        } else {
            ...
            // refund any excess ETH to the caller
            if (msg.value > netInputAmount) msg.sender.safeTransferETH(msg.value - netInputAmount);
        }
        ...
    }
```

## [04] `Factory.setProtocolFeeRate` FUNCTION DOES NOT HAVE A LIMIT FOR SETTING `protocolFeeRate`
The following `Factory.setProtocolFeeRate` function can be called to set `protocolFeeRate` to be more than 10_000, which is unlike the `PrivatePool.setFeeRate` function that limits `feeRate` to be less or equal to 5_000. Setting `protocolFeeRate` without a limit can have unexpected consequences. For example, when `protocolFeeRate` is set to more than 10_000, the protocol fee can even exceed the total of all NFTs' `salePrice` when calling the `PrivatePool.buy` function in which the cost can be too high to the user unexpectedly.

As a mitigation, the `Factory.setProtocolFeeRate` function can be updated to only set `protocolFeeRate` to a value that cannot exceed certain reasonable limit, such as 5_000.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L141-L143
```solidity
    function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner {
        protocolFeeRate = _protocolFeeRate;
    }
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L562-L571
```solidity
    function setFeeRate(uint16 newFeeRate) public onlyOwner {
        // check that the fee rate is less than 50%
        if (newFeeRate > 5_000) revert FeeRateTooHigh();

        // set the fee rate
        feeRate = newFeeRate;

        // emit the set fee rate event
        emit SetFeeRate(newFeeRate);
    }
```

## [05] ETH AMOUNTS CAN BE SENT TO `EthRouter` AND `Factory` CONTRACTS ACCIDENTALLY
Because of the following `EthRouter.receive` and `Factory.receive` functions, ETH amounts can be accidentally sent to the `EthRouter` and `Factory` contracts. To prevent users accidentally send and lose such ETH amounts, the `EthRouter.receive` and `Factory.receive` functions can be updated to check if `msg.sender` is the `PrivatePool` contract; if not, calling these function should revert.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L88
```solidity
    receive() external payable {}
```

https://github.com/code-423n4/2023-04-caviar/blob/main/src/Factory.sol#L55
```solidity
    receive() external payable {}
```

## [06] MISSING `address(0)` CHECKS FOR CRITICAL ADDRESS INPUTS
To prevent unintended behaviors, the critical address inputs should be checked against `address(0)`. 

The `address(0)` check is missing for the `_royaltyRegistry` input variable in the following constructor. Please consider checking it.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/EthRouter.sol#L90-L92
```solidity
    constructor(address _royaltyRegistry) {
        royaltyRegistry = _royaltyRegistry;
    }
```

The `address(0)` checks are missing for the `_factory`, `_royaltyRegistry`, and `_stolenNftOracle` input variables in the following constructor. Please consider checking them.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePool.sol#L143-L147
```solidity
    constructor(address _factory, address _royaltyRegistry, address _stolenNftOracle) {
        factory = payable(_factory);
        royaltyRegistry = _royaltyRegistry;
        stolenNftOracle = _stolenNftOracle;
    }
```

## [07] CASTING `factory` TO `address` IS REDUNDANT
In the following code, `factory` is already `address` so it is redundant to cast it again to `address`. To make the code more efficient, please consider directly using `factory` instead of `address(factory)`.

```solidity
src\PrivatePool.sol
  259: if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
  368: if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
```

## [08] CONSTANTS CAN BE USED INSTEAD OF MAGIC NUMBERS
To improve readability and maintainability, a constant can be used instead of the magic number. Please consider replacing the magic numbers, such as `10_000`, used in the following code with constants.

```solidity
src\PrivatePool.sol
  172: if (_feeRate > 5_000) revert FeeRateTooHigh();  
  668: return tokenIds.length * 1e18;  
  703: protocolFeeAmount = inputAmount * Factory(factory).protocolFeeRate() / 10_000;  
  704: feeAmount = inputAmount * feeRate / 10_000;  
  721: protocolFeeAmount = outputAmount * Factory(factory).protocolFeeRate() / 10_000;  
  722: feeAmount = outputAmount * feeRate / 10_000;  
  734: uint256 feePerNft = changeFee * 10 ** exponent; 
  736: feeAmount = inputAmount * feePerNft / 1e18; 
  737: protocolFeeAmount = feeAmount * Factory(factory).protocolFeeRate() / 10_000; 
```

## [09] FLOATING PRAGMAS
It is a best practice to lock pragmas instead of using floating pragmas to ensure that contracts are tested and deployed with the intended compiler version. Accidentally deploying contracts with different compiler versions can lead to unexpected risks and undiscovered bugs. Please consider locking pragmas for the following files.

```solidity
src\EthRouter.sol
  2: pragma solidity ^0.8.19;    

src\Factory.sol
  2: pragma solidity ^0.8.19;    

src\PrivatePool.sol
  2: pragma solidity ^0.8.19;    

src\PrivatePoolMetadata.sol
  2: pragma solidity ^0.8.19;    
```

## [10] INCOMPLETE NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following functions miss the `@param` or `@return` comments. Please consider completing the NatSpec comments for these functions.

```solidity
src\EthRouter.sol
  301: function getRoyalty(address nft, uint256 tokenId, uint256 salePrice)    

src\Factory.sol
  71: function create(

src\PrivatePool.sol
  157: function initialize(    
  211: function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
  301: function sell(  
  385: function change(    
  694: function buyQuote(uint256 outputAmount) 
  713: function sellQuote(uint256 inputAmount) 

src\PrivatePoolMetadata.sol
  17: function tokenURI(uint256 tokenId) public view returns (string memory) {    
  35: function attributes(uint256 tokenId) public view returns (string memory) {    
  55: function svg(uint256 tokenId) public view returns (bytes memory) {  
```

## [11] MISSING NATSPEC COMMENTS
NatSpec comments provide rich code documentation. The following function miss NatSpec comments. Please consider adding NatSpec comments for this function.

https://github.com/code-423n4/2023-04-caviar/blob/main/src/PrivatePoolMetadata.sol#L112-L119
```solidity
    function trait(string memory traitType, string memory value) internal pure returns (string memory) {
        // forgefmt: disable-next-item
        return string(
            abi.encodePacked(
                '{ "trait_type": "', traitType, '",', '"value": "', value, '" }'
            )
        );
    }
```