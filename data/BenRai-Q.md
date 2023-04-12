
# Add a Timelock to Critical Parameter Change

It is a good practice to give time for users to react and adjust to critical changes with a mandatory time window between them. The first step merely broadcasts to users that a particular change is coming, and the second step commits that change after a suitable waiting period. This allows users that do not accept the change to withdraw within the grace period. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious Owner making any malicious or ulterior intention). Specifically, privileged roles could use front running to make malicious changes just ahead of incoming transactions, or purely accidental negative effects could occur due to the unfortunate timing of changes.

Considder implementing a time look feature for chritical parameter that influance the calculation of the amound a user needs to pay to the pool or reseives from the pool like the following:

### Factory.sol:
function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner

### PrivatePool.sol:

function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner
function setFeeRate(uint16 newFeeRate) public onlyOwner
function setPayRoyalties(bool newPayRoyalties) public onlyOwner


# Events are indexing to little fields

Indexing event fields makes the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it’s not necessarily best to index the maximum allowed per event (threefields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed. 


There are 12 instenses of this issue:

### Factory.sol
 event Create(address indexed privatePool, uint256[] tokenIds, uint256 baseTokenAmount);  


### PrivatePool.sol

event Initialize(address indexed baseToken, address indexed nft, uint128 virtualBaseTokenReserves, uint128 virtualNftReserves, uint56 changeFee, uint16 feeRate, bytes32 merkleRoot, bool useStolenNftOracle, bool payRoyalties);
    event Buy(uint256[] tokenIds, uint256[] tokenWeights, uint256 inputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
    event Sell(uint256[] tokenIds, uint256[] tokenWeights, uint256 outputAmount, uint256 feeAmount, uint256 protocolFeeAmount, uint256 royaltyFeeAmount);
    event Deposit(uint256[] tokenIds, uint256 baseTokenAmount);
    event Withdraw(address indexed nft, uint256[] tokenIds, address token, uint256 amount);
    event Change(uint256[] inputTokenIds, uint256[] inputTokenWeights, uint256[] outputTokenIds, uint256[] outputTokenWeights, uint256 feeAmount, uint256 protocolFeeAmount);
    event SetVirtualReserves(uint128 virtualBaseTokenReserves, uint128 virtualNftReserves);
    event SetMerkleRoot(bytes32 merkleRoot);
    event SetFeeRate(uint16 feeRate);
    event SetUseStolenNftOracle(bool useStolenNftOracle);
    event SetPayRoyalties(bool payRoyalties);


# Import of IERC3156FlashBorrower should come from IERC3156FlashBorrower.sol not from IERC3156FlashLender.sol

As a good practice an interface Import should come from the main file and not from a file that imports the interface itselve:

### file src/PrivatePool.sol
import {IERC3156FlashBorrower} from "openzeppelin/interfaces/IERC3156FlashLender.sol"


# Missing comment about address == 0 for the base token address 
Use a comment to make it clear that if the base token address is 0, ETH is used as a base token. This would make the code easier to understand.


# Missing check for address(0x0) when assigning values to address state variables
When assigning values to address state variables there is no check for address(0x0). This can be especially problematic in the constructor of PrivatePool.sol since the address state variables for factory, royaltyRegistry and stolenNftOracle are immutable and can not be changed. This can lead to lose of functionality in the case of a wrong royaltyRegistry and stolenNftOracle or even to lose of the protocol fees for the factory owner in case of the factory address. Loosing functionality might lead to the need of redeploying the pool and therefore to additional costs for removing all NFTs and baseTokens from the pool and redeploying the pool the right way.  

# Typo in comment (“aount” should be “amount”)
### file src/PrivatePool.sol
252: // add the royalty fee amount to the net input aount

# Typo in comment (“the” should be removed)
### file src/PrivatePool.sol
574: Sets the whether or not to use the stolen NFT oracle

# unreachable Code
Since the royalties are in % of the sales price they can never be higher than the sales price. Check and definition of custom error can be removed for lower deployment gas
### file src/PrivatePool.sol
801:  if (royaltyFee > salePrice) revert InvalidRoyaltyFee();
79:     error InvalidRoyaltyFee();


# No tracking of the accumulated fees a pool has earned
There is no tracking of the fees the pool has earned and since the owner can set the virtualBaseTokenReserves freely without been restricted by the base tokens held by the pool there is no way to calculate the fees earned so far. (balanceOfContract – virtualBaseTokenReserves can not be used to calculate the fees earned). This makes it inconvenient for the owner of the pool to withdraw the earned fees. I would suggest to add a variable that tracks the fees earned by the pool and that is set to zero when the owner withdraws equal or more than the fees earned.

# Make immutable variables changeable
The variables of the addresses stolenNftOracle and  royaltyRegistry are immutable. In case the addresses of those two variables need to be changed (eg. because one needs to use an other provider of this service due to the current service not working any more) there is no possibility to do so. In this case the owner would need to withdraw all funds/nfts from the pool and redeploy it with the new addresses and the address of the pool would change. Therefore I suggest removing the immutable keyword for this variables and adding setter functions for both to be more flexible.


# Change the calculation of the fees for the function change() 
The fees for the function change() are calculated based on the inputweight of the NFTs the user is putting into the pool.
          414: (feeAmount, protocolFeeAmount) = changeFeeQuote(inputWeightSum);
This is contra intuitive and can lead to a bad user experience. If the user puts in NFTs that are more valuable than the NFTs he gets out the pool already profits from that. If the user in addition has to pay fees based on the value of the NFT he puts in he is disadvantaged twice. Therefore I would suggest to base the calculation of the fees for the change() function on the outputWeightSum not the inputWeightSum.


# There is no check if the contract owns the NFTs or not
For the functions buy(), sell(), change(), deposit() and withdraw() there is no check if the alleged current owner really owns the NFTs in the tokenIds array. To save gas in case one of the Ids is not owned by the alleged owner (what would result in the whole transaction to fail) I suggest adding a check if the  alleged owner really owns the Nft and if not fail the transaction early. Such a check is already implemented in the function flashLoan() with the call of the function availableForFlashLoan().

