# Add a Timelock to Critical Parameter Change

It is a good practice to give time for users to react and adjust to critical changes with a mandatory time window between them. The first step merely broadcasts to users that a particular change is coming, and the second step commits that change after a suitable waiting period. This allows users that do not accept the change to withdraw within the grace period. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious Owner making any malicious or ulterior intention). Specifically, privileged roles could use front running to make malicious changes just ahead of incoming transactions, or purely accidental negative effects could occur due to the unfortunate timing of changes.

Considder implementing a time look feature for chritical parameter that influance the calculation of the amound a user needs to pay to the pool or reseives from the pool like the following:

### Factory.sol:
function setProtocolFeeRate(uint16 _protocolFeeRate) public onlyOwner

### PrivatePool.sol:

function setVirtualReserves(uint128 newVirtualBaseTokenReserves, uint128 newVirtualNftReserves) public onlyOwner
function setFeeRate(uint16 newFeeRate) public onlyOwner
function setPayRoyalties(bool newPayRoyalties) public onlyOwner
