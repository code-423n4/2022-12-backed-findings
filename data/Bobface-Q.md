Papr QA Report
===

The codebase looks well structured and adheres to best practices throughout. A few minor possible improvements are listed below.

## Non-Critical

### Specify variable visibility
The visibility of variables should be explicitly stated, even if the default value is intended.
| File                    | Lines                         | Description |
|---|---|---|
| `src/PaprToken.sol` | 9 | `controller` |
| `src/ReservoirOracleUnderwriter.sol` | 35 | `TWAP_SECONDS` |
| `src/ReservoirOracleUnderwriter.sol` | 38 | `VALID_FOR` |

### Avoid duplicate code
Avoid duplicate code blocks.
| File                    | Lines                         | Description |
|---|---|---|
| `src/PaprController.sol` | 182 - 205, 493 - 517| Use `memory` on the first function and remove the second function. Removing duplicate logic is more important than saving the little amount of gas by the two different implementation for `calldata` and `memory` parameters. |


### Use built-in `type(...).max` value
When checking that a value does not overflow on conversion to a smaller type, use `type(...).max` instead of calculating the maximum value through arithmetic.
| File                    | Lines                         | Description |
|---|---|---|
| `src/PaprController.sol` | 473 | `newDebt >= 1 << 200` to `newDebt > type(uint200).max` |

### Prefer `constant` over `immutable`
Constant variables which are not changed in the `constructor` should be `constant` instead of `immutable`.
| File                    | Lines                         | Description |
|---|---|---|
| `src/PaprController.sol` | 41 | `liquidationAuctionMinSpacing` |
| `src/PaprController.sol` | 44 | `perPeriodAuctionDecayWAD` |
| `src/PaprController.sol` | 47 | `auctionDecayPeriod` |
| `src/PaprController.sol` | 50 | `auctionStartPriceMultiplier` |
| `src/PaprController.sol` | 54 | `liquidationPenaltyBips` |

### Store magic numbers as `constant`s
Magic numbers should be stored as `constant`s and given an appropriate name.
| File                    | Lines                         | Description |
|---|---|---|
| `src/PaprController.sol` | 87, 89 | `10 ** 18` |
| `src/PaprController.sol` | 92 | `10000` |
| `src/UniswapOracleFundingRateController.sol` | 138 | `1e18` |

### Format large numbers with underscores
Large numbers should be formatted with underscores to improve readability, e.g. `1000000` -> `1_000_000`.
| File                    | Lines                         | Description |
|---|---|---|
| `src/PaprController.sol` | 54 | `1000` |
| `src/PaprController.sol` | 92 | `10000` |