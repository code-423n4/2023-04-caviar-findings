## PrivatePool

1- `baseToken`, `nft`, and `changeFee` cannot change throughout a private pool's lifecycle so consider moving them to be set in the constructor and change them to be `immutable`