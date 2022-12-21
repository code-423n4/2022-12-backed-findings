# Gas Optimization Report

## 1. src/NFTEDA/libraries/EDAPrice.sol#L17

Variable `perPeriodDecayPercentWad` seems only being used in the `currentPrice` function, and will be used to calculate `percentWadRemainingPerPeriod` by doing a subtraction each time. Simply storing `percentWadRemainingPerPeriod` in the `Auction` struct will make this function more gas-efficient

## 2. src/PaprController.sol#L321

This check of `lastAuctionStartTime` can be `unchecked`, since `block.timestamp` is always bigger or equal.
