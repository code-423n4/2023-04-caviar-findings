Files analyzed:
- src/EthRouter.sol
- src/Factory.sol
- src/PrivatePool.sol
- src/PrivatePoolMetadata.sol
- src/interfaces/IStolenNftOracle.sol

Issues found:
 G001:
  src/EthRouter.sol::106 => for (uint256 i = 0; i < buys.length; i++) {
  src/EthRouter.sol::116 => for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
  src/EthRouter.sol::134 => for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
  src/EthRouter.sol::159 => for (uint256 i = 0; i < sells.length; i++) {
  src/EthRouter.sol::161 => for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
  src/EthRouter.sol::183 => for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
  src/EthRouter.sol::239 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/EthRouter.sol::261 => for (uint256 i = 0; i < changes.length; i++) {
  src/EthRouter.sol::265 => for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
  src/EthRouter.sol::284 => for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
  src/Factory.sol::119 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::237 => uint256 royaltyFeeAmount = 0;
  src/PrivatePool.sol::238 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::272 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::328 => uint256 royaltyFeeAmount = 0;
  src/PrivatePool.sol::329 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::441 => for (uint256 i = 0; i < inputTokenIds.length; i++) {
  src/PrivatePool.sol::446 => for (uint256 i = 0; i < outputTokenIds.length; i++) {
  src/PrivatePool.sol::496 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::518 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::673 => for (uint256 i = 0; i < tokenIds.length; i++) {

 G002:
  src/EthRouter.sol::106 => for (uint256 i = 0; i < buys.length; i++) {
  src/EthRouter.sol::115 => uint256 salePrice = inputAmount / buys[i].tokenIds.length;
  src/EthRouter.sol::116 => for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
  src/EthRouter.sol::134 => for (uint256 j = 0; j < buys[i].tokenIds.length; j++) {
  src/EthRouter.sol::159 => for (uint256 i = 0; i < sells.length; i++) {
  src/EthRouter.sol::161 => for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
  src/EthRouter.sol::182 => uint256 salePrice = outputAmount / sells[i].tokenIds.length;
  src/EthRouter.sol::183 => for (uint256 j = 0; j < sells[i].tokenIds.length; j++) {
  src/EthRouter.sol::239 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/EthRouter.sol::261 => for (uint256 i = 0; i < changes.length; i++) {
  src/EthRouter.sol::265 => for (uint256 j = 0; j < changes[i].inputTokenIds.length; j++) {
  src/EthRouter.sol::284 => for (uint256 j = 0; j < changes[i].outputTokenIds.length; j++) {
  src/Factory.sol::119 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::236 => uint256 salePrice = (netInputAmount - feeAmount - protocolFeeAmount) / tokenIds.length;
  src/PrivatePool.sol::238 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::272 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::329 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::335 => uint256 salePrice = (netOutputAmount + feeAmount + protocolFeeAmount) / tokenIds.length;
  src/PrivatePool.sol::441 => for (uint256 i = 0; i < inputTokenIds.length; i++) {
  src/PrivatePool.sol::446 => for (uint256 i = 0; i < outputTokenIds.length; i++) {
  src/PrivatePool.sol::467 => if (returnData.length > 0) {
  src/PrivatePool.sol::496 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::518 => for (uint256 i = 0; i < tokenIds.length; i++) {
  src/PrivatePool.sol::668 => return tokenIds.length * 1e18;
  src/PrivatePool.sol::672 => bytes32[] memory leafs = new bytes32[](tokenIds.length);
  src/PrivatePool.sol::673 => for (uint256 i = 0; i < tokenIds.length; i++) {

 G003:
  src/EthRouter.sol::121 => if (royaltyFee > 0) {
  src/EthRouter.sol::141 => if (address(this).balance > 0) {
  src/EthRouter.sol::188 => if (royaltyFee > 0) {
  src/EthRouter.sol::290 => if (address(this).balance > 0) {
  src/Factory.sol::87 => if ((_baseToken == address(0) && msg.value != baseTokenAmount) || (_baseToken != address(0) && msg.value > 0)) {
  src/PrivatePool.sol::225 => if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
  src/PrivatePool.sol::259 => if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
  src/PrivatePool.sol::265 => if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
  src/PrivatePool.sol::277 => if (royaltyFee > 0 && recipient != address(0)) {
  src/PrivatePool.sol::344 => if (royaltyFee > 0 && recipient != address(0)) {
  src/PrivatePool.sol::362 => if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
  src/PrivatePool.sol::368 => if (protocolFeeAmount > 0) ERC20(baseToken).safeTransfer(address(factory), protocolFeeAmount);
  src/PrivatePool.sol::397 => if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();
  src/PrivatePool.sol::426 => if (protocolFeeAmount > 0) ERC20(baseToken).safeTransferFrom(msg.sender, factory, protocolFeeAmount);
  src/PrivatePool.sol::432 => if (protocolFeeAmount > 0) factory.safeTransferETH(protocolFeeAmount);
  src/PrivatePool.sol::467 => if (returnData.length > 0) {
  src/PrivatePool.sol::489 => if ((baseToken == address(0) && msg.value != baseTokenAmount) || (msg.value > 0 && baseToken != address(0))) {

 G006:
  src/PrivatePool.sol::642 => receiver.onFlashLoan(msg.sender, token, tokenId, fee, data) == keccak256("ERC3156FlashBorrower.onFlashLoan");
  src/PrivatePool.sol::675 => leafs[i] = keccak256(bytes.concat(keccak256(abi.encode(tokenIds[i], tokenWeights[i]))));

 G007:
  src/EthRouter.sol::32 => import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
  src/EthRouter.sol::33 => import {IERC2981} from "openzeppelin/interfaces/IERC2981.sol";
  src/EthRouter.sol::35 => import {IRoyaltyRegistry} from "royalty-registry-solidity/IRoyaltyRegistry.sol";
  src/EthRouter.sol::38 => import {IStolenNftOracle} from "./interfaces/IStolenNftOracle.sol";
  src/Factory.sol::27 => import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
  src/PrivatePool.sol::29 => import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
  src/PrivatePool.sol::30 => import {SafeTransferLib} from "solmate/utils/SafeTransferLib.sol";
  src/PrivatePool.sol::32 => import {IERC2981} from "openzeppelin/interfaces/IERC2981.sol";
  src/PrivatePool.sol::33 => import {IRoyaltyRegistry} from "royalty-registry-solidity/IRoyaltyRegistry.sol";
  src/PrivatePool.sol::34 => import {IERC3156FlashBorrower} from "openzeppelin/interfaces/IERC3156FlashLender.sol";
  src/PrivatePool.sol::36 => import {IStolenNftOracle} from "./interfaces/IStolenNftOracle.sol";
  src/PrivatePoolMetadata.sol::21 => '"name": "Private Pool ',Strings.toString(tokenId),'",',
  src/PrivatePoolMetadata.sol::22 => '"description": "Caviar private pool AMM position.",',
  src/PrivatePoolMetadata.sol::23 => '"image": ','"data:image/svg+xml;base64,', Base64.encode(svg(tokenId)),'",',
  src/PrivatePoolMetadata.sol::63 => '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 400 400" style="width:100%;background:black;fill:white;font-family:serif;">',
  src/PrivatePoolMetadata.sol::116 => '{ "trait_type": "', traitType, '",', '"value": "', value, '" }'

 G008:
  src/PrivatePoolMetadata.sol::63 => '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 400 400" style="width:100%;background:black;fill:white;font-family:serif;">',

 L001:
  src/Factory.sol::115 => ERC20(_baseToken).transferFrom(msg.sender, address(privatePool), baseTokenAmount);
  src/Factory.sol::152 => ERC20(token).transfer(msg.sender, amount);
  src/PrivatePool.sol::365 => ERC20(baseToken).transfer(msg.sender, netOutputAmount);
  src/PrivatePool.sol::527 => ERC20(token).transfer(msg.sender, tokenAmount);
  src/PrivatePool.sol::651 => if (baseToken != address(0)) ERC20(baseToken).transferFrom(msg.sender, address(this), fee);

 L003:
  src/EthRouter.sol::2 => pragma solidity ^0.8.19;
  src/Factory.sol::2 => pragma solidity ^0.8.19;
  src/PrivatePool.sol::2 => pragma solidity ^0.8.19;
  src/PrivatePoolMetadata.sol::2 => pragma solidity ^0.8.19;
  src/interfaces/IStolenNftOracle.sol::2 => pragma solidity ^0.8.19;


