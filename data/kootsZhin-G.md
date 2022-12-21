# Gas optimisation report

## [G-01] Use `constant` instead of `immutable` for hard-coded value
For "constant" variables that are hard-coded into the contract without the need to be initialized in the constrcutor, use `constant` instead of `immutable` to save deployment cost.

_instance (5)_:
```solidity
file: src/PaprController.sol, line 40 - 54

    /// @inheritdoc IPaprController
    uint256 public immutable override liquidationAuctionMinSpacing = 2 days;

    /// @inheritdoc IPaprController
    uint256 public immutable override perPeriodAuctionDecayWAD = 0.7e18;

    /// @inheritdoc IPaprController
    uint256 public immutable override auctionDecayPeriod = 1 days;

    /// @inheritdoc IPaprController
    uint256 public immutable override auctionStartPriceMultiplier = 3;

    /// @inheritdoc IPaprController
    /// @dev Set to 10%
    uint256 public immutable override liquidationPenaltyBips = 1000;
```
