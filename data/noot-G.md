#  _purchaseNFTAndUpdateVaultIfNeeded should be called first in purchaseLiquidationAuctionNFT

In `purchaseLiquidationAuctionNFT()`, various calculations are done before `_purchaseNFTAndUpdateVaultIfNeeded()` is called. However, none of these calculated values are passed to `_purchaseNFTAndUpdateVaultIfNeeded()`, which may revert. The call to `_purchaseNFTAndUpdateVaultIfNeeded()` should be the first thing to happen in the function to save users gas in case of reverts.

# `removeCollateral` should accept accept ERC721 address and array of IDs instead of array of Collateral

This function requires all collateral addresses to be the same. To remove the address check which happens on every iteration >0, this function should instead accept the collateral address and array of IDs. This can also remove the `if (i == 0)` check.

# timestamp check in `startLiquidationAuction` is unneeded

There is no way to end an auction once it starts (apart from purchasing the NFT), thus calling `startLiquidationAuction` multiple times will revert in the `_startAuction` call. The timestamp check inside the function:
```
        if (block.timestamp - info.latestAuctionStartTime < liquidationAuctionMinSpacing) {
            revert IPaprController.MinAuctionSpacing();
        }
```
effectively does nothing, as `info.latestAuctionStartTime` will always be 0 on the first call to start the auction, and if it's set, the call will revert during `_startAuction`.

# `newDebt` check should be executed earlier in `_increaseDebt()`

The check for `newDebt` inside `_increaseDebt` is as follows:

```
        if (newDebt >= 1 << 200) revert IPaprController.DebtAmountExceedsUint200();
```

This line should be moved directly under the line which calculates `newDebt` to save gas in case of reverts.