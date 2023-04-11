## [L-01] Users can't create pools while setPrivatePoolImplementation() didn't called by owner.
SC: Factory.sol

## Proof of Concept
Users can't create pools while setPrivatePoolImplementation() didn't called by owner.
https://github.com/code-423n4/2023-04-caviar/blob/cd8a92667bcb6657f70657183769c244d04c015c/src/Factory.sol#L135

## Recommended Mitigation Steps
To avoid situation where users can't create pools it is necessary to set poolImplemmentation in factory constructor.

## [L-02] Unsafe transferOwnership() by solmate.
SC: Factory.sol with imported Owned.sol by solmate lib.

## Proof of Concept
The function transferOwnership() transfer ownership to a new address. In case a wrong address ownership will be permanently lose.


## Recommended Mitigation Steps

There is another Openzeppelin Ownable contract (Ownable2StepUpgradeable.sol) has transferOwnership function, use is more secure due to 2-stage ownership transfer.
![Tux, the Linux mascot](https://i.imgur.com/buDGgQr.png)

## [L-03] deposit() does not has `onlyOwner` modifier.
SC: PrivatePool.sol

## Proof of Concept
Despite the fact that the owner mainly will use the function, it is likely that it will be called by an ordinary user due to the lack of `onlyOwner` modifier. There is no any rewards for whom who made deposit, no LPs, and no withdraw option for users.

## Recommended Mitigation Steps
Add `onlyOwner` modifier.