## [L-01] Functions should return value but they don't
## Details
Functions on Pair.sol, wrap(), nftSell(), unwrap(), nftRemove(), nftAdd(), sell(), buy(), remove(), add(), do not return any value at the end
## Code snipets
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L70-L77
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L134-L139
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L180-L183
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L223-L225
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L264-L266
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L303
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L345-L354
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L370-L376
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L390-L393
https://github.com/outdoteth/caviar/blob/1065267078876005b4743f5705d5426f98f80da0/src/Pair.sol#L408-L414
## Recommendations
function add( uint256 baseTokenAmount, uint256 fractionalTokenAmount, uint256 minLpTokenAmount, uint256 minPrice, uint256 maxPrice, uint256 deadline ) public payable returns (uint256 lpTokenAmount) {
…
return lpTokenAmount;
}

function remove( uint256 lpTokenAmount, uint256 minBaseTokenOutputAmount, uint256 minFractionalTokenOutputAmount, uint256 deadline ) public returns (uint256 baseTokenOutputAmount, uint256 fractionalTokenOutputAmount) {
…
return (baseTokenOutputAmount, fractionalTokenOutputAmount);
}  
  
function buy(uint256 outputAmount, uint256 maxInputAmount, uint256 deadline) public payable returns (uint256 inputAmount) {
…
return inputAmount;
}

function sell(uint256 inputAmount, uint256 minOutputAmount, uint256 deadline) public returns (uint256 outputAmount) {
…
return outputAmount;
}

function wrap(uint256[] calldata tokenIds, bytes32[][] calldata proofs, ReservoirOracle.Message[] calldata messages) public returns (uint256 fractionalTokenAmount) {
…
return fractionalTokenAmount;
}

function unwrap(uint256[] calldata tokenIds, bool withFee) public returns (uint256 fractionalTokenAmount) {
…
return fractionalTokenAmount;
}

function nftAdd( uint256 baseTokenAmount, uint256[] calldata tokenIds, uint256 minLpTokenAmount, uint256 minPrice, uint256 maxPrice, uint256 deadline, bytes32[][] calldata proofs, ReservoirOracle.Message[] calldata messages ) public payable returns (uint256 lpTokenAmount) {
…
return lpTokenAmount = add(baseTokenAmount, fractionalTokenAmount, minLpTokenAmount, minPrice, maxPrice, deadline);
}

function nftRemove( uint256 lpTokenAmount, uint256 minBaseTokenOutputAmount, uint256 deadline, uint256[] calldata tokenIds, bool withFee ) public returns (uint256 baseTokenOutputAmount, uint256 fractionalTokenOutputAmount) {
…
return (baseTokenOutputAmount, fractionalTokenOutputAmount);
}

function nftBuy(uint256[] calldata tokenIds, uint256 maxInputAmount, uint256 deadline) public payable returns (uint256 inputAmount) {
…
return inputAmount;
}

function nftSell( uint256[] calldata tokenIds, uint256 minOutputAmount, uint256 deadline, bytes32[][] calldata proofs, ReservoirOracle.Message[] calldata messages ) public returns (uint256 outputAmount) {
…
return outputAmount;
}
