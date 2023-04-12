## 01. In some cases, use nft.transferFrom() instead of nft.safeTransferFrom()
In PrivatePool.sol there is no logic onErc721Received. To avoid gas spending on that call, don't call it.
In places where destination of nft trandfer is PrivatePool.sol, use nft.transferFrom()