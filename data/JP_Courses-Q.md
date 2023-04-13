0)
https://github.com/code-423n4/2023-04-caviar/blob/2cfa974fc146acb9dc53b7d410222483f4b15dd7/src/EthRouter.sol#L219
https://github.com/code-423n4/2023-04-caviar/blob/463473a74640f2bf5907c0376959f1215e861fbc/src/PrivatePool.sol#L484
/// @audit LOW/MEDIUM?: right now only the pool owner can deposit & withdraw NFTs & basetokens, i.e. add/remove liquidity. 

If this is intentional, the documentation doesnt make this clear at all. In fact, there are places in the documentation that imply that private pool users will be able to add/deposit liquidity to the pools too, and also earn yield from providing liquidity. Frontend GUI only has deposit functionality for shared pools.
(Should probably have a modifier added to both the deposit & withdraw functions that only allow private pool owners to access those functions.)
So, as long as protocol's intention is to make liquidity deposits/withdrawals only available to private pool owners, then all good, except for one thing: Please check my medium risk number 0) for more details.

1)
https://github.com/code-423n4/2023-04-caviar/blob/2cfa974fc146acb9dc53b7d410222483f4b15dd7/src/EthRouter.sol#L238-L241
/// @audit LOW: shouldn't we check first that the NFTs are not already in the router, in case transaction is interrupted before router deposits NFTs to private pool...

/// @audit fixed:
/// @audit add before the deposit function:

mapping(address => mapping(uint256 => bool)) private nftsInRouter;

/// @audit (add before the for loop for ERC721(nft).safeTransferFrom):

    // Check if NFTs are already in the router
    uint256 tokenIdsLength = tokenIds.length;
    for unchecked(uint256 i = 0; i < tokenIdsLength; i++) {
    		uint256 tokenId = tokenIds[i];
        require(!nftsInRouter[nft][tokenId], "addLiquidityNFT: NFT already in the router");
        nftsInRouter[nft][tokenId] = true;
    }
    
2) 
/// @audit LOW: no require check to ensure that the NFTs were successfully transferred

// transfer NFTs from caller
for (uint256 i = 0; i < tokenIds.length; i++) {
    uint256 tokenId = tokenIds[i];
    ERC721(nft).safeTransferFrom(msg.sender, address(this), tokenId);
    require(ERC721(nft).ownerOf(tokenId) == address(this), "NFT transfer failed"); /// @audit fixed
}

3)
https://github.com/code-423n4/2023-04-caviar/blob/2cfa974fc146acb9dc53b7d410222483f4b15dd7/src/EthRouter.sol#L247
/// @audit LOW: shouldn't we check that the deposit of the NFTs(and ETH) to the private pool were successful?

4)
https://github.com/code-423n4/2023-04-caviar/blob/2cfa974fc146acb9dc53b7d410222483f4b15dd7/src/EthRouter.sol#L219
/// @audit LOW: shouldn't there be checks to ensure that values for parameters privatePool & nft are valid addresses != address(0)
/// @audit LOW: and also to check length of tokenIds != 0

5)
https://github.com/code-423n4/2023-04-caviar/blob/463473a74640f2bf5907c0376959f1215e861fbc/src/PrivatePool.sol#L393
/// @audit LOW: shouldn't this change() function be callable ONLY from the EthRouter's change() function? 

Unsure what happens if this function is called directly by a user/attacker if they can input custom values for the parameters, including address value for nft variable.
So, I suggest to add a modifier that checks that the caller is EthRouter address. 

6)
https://github.com/code-423n4/2023-04-caviar/blob/463473a74640f2bf5907c0376959f1215e861fbc/src/PrivatePool.sol#L440-L443
/// @audit LOW: should be a check to ensure input NFTs were received into private pool, before transferring output NFTs to the EthRouter contract.

7) 
https://github.com/code-423n4/2023-04-caviar/blob/463473a74640f2bf5907c0376959f1215e861fbc/src/PrivatePool.sol#L509-L532
/// @audit LOW: no check to ensure that the token address parameter is equal to the basetoken of the pool. 

Which means owner can withdraw all _nft collection NFTs from his pool before failing to withdraw any ETH, if he selected the ETH address as token parameter but ERC20 token is basetoken for pool. Likewise if owner selected ERC20 for token parameter, but actually ETH in pool as basetoken.
It will withdraw the NFTs before it fails to withdraw the basetoken's tokenAmount.

8)
/// @audit LOW: the pool owner can only withdraw ALL NFTs from the pool in one withdraw transaction if all the NFTs are from the same collection. 

i.e. _nft parameter value is the same for all the pool's NFTs. Otherwise pool owner will need to execute multiple withdraw transactions.
Is this intended functionality? What will the pool owner understand the withdraw function to do? Will pool owner's assumption be in line with protocol's intention?

9)
https://github.com/code-423n4/2023-04-caviar/blob/2cfa974fc146acb9dc53b7d410222483f4b15dd7/src/EthRouter.sol#L250-L293
https://github.com/code-423n4/2023-04-caviar/blob/463473a74640f2bf5907c0376959f1215e861fbc/src/PrivatePool.sol#L375-L452
/// @audit MEDIUM: it's clear that the change() function is designed to change NFTs ONLY from the same NFT collection, so it's definitely not possible to change a user's NFT to an NFT from another collection/address. 

For both input & output NFTs, it uses the same Change struct instance and the same value/address for the nft variable inside the struct. Is this intentional? If the idea was to be able to also change from an NFT in collection A with addressA, to NFT from collection B with addressB, then this is currently not possible. I assume the frontend has a check for this? If not, then the user can select NFT from another collection to change, then when he calls the change function, the NFT id from the other collection will be used to give him the NFT from the same collection of his current NFT, and with the same NFT id as from the other collection that he selected. I did not check the frontend due to insufficient testnet funds.
Frontend might have the correct checks in place, including the presence/lack of functionality as per protocol intentions, but the "backend" smart contracts dont always reflect this. An attacker can bypass the frontend checks easily.

Should be made clearer on both frontend and backend, including for pool owner & users that the change function currently supports only changing between NFTs from same collection.

10)
https://github.com/code-423n4/2023-04-caviar/blob/2cfa974fc146acb9dc53b7d410222483f4b15dd7/src/EthRouter.sol#L272-L286
/// @audit MEDIUM: there should be a check after the PrivatePool(_change.pool).change to ensure that EthRouter received the outputToken NFT, before calling ERC721(_change.nft).safeTransferFrom to transfer the outputToken NFT to the user.

11)
https://github.com/code-423n4/2023-04-caviar/blob/463473a74640f2bf5907c0376959f1215e861fbc/src/PrivatePool.sol#L478-L507
https://github.com/code-423n4/2023-04-caviar/blob/2cfa974fc146acb9dc53b7d410222483f4b15dd7/src/EthRouter.sol#L211-L248
/// @audit HIGH: It seems possible that an attacker could "inject" a rogue NFT into the private pool by calling the PrivatePool's deposit function directly:

call this deposit function directly and use a rogue contract address for nft parameter and use the rogue NFT to exploit one or more private pools whenever pool owner or users interact with this rogue NFT's contract?
