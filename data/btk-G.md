## G-1: immutable variables that aren't declared in the constructor should be declared constant instead

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L41-L54

## Before:

```solidity
uint256 public immutable override liquidationAuctionMinSpacing = 2 days;
```
```solidity
uint256 public immutable override perPeriodAuctionDecayWAD = 0.7e18;
```
```solidity
uint256 public immutable override auctionDecayPeriod = 1 days;
```
```solidity
uint256 public immutable override auctionStartPriceMultiplier = 3;
```
```solidity
uint256 public immutable override liquidationPenaltyBips = 1000;
```

| src/PaprController.sol:PaprController contract |                 |        |        |        |         |
|------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                | Deployment Size |        |        |        |         |
| 10374332                                       | 36526           |        |        |        |         |
| Function Name                                  | min             | avg    | median | max    | # calls |
| addCollateral                                  | 3236            | 66947  | 79852  | 94442  | 26      |
| auctionCurrentPrice                            | 4713            | 6141   | 6713   | 6713   | 7       |
| burnPaprFromAuctionFees                        | 2620            | 3663   | 3663   | 4707   | 2       |
| buyAndReduceDebt                               | 57308           | 59758  | 59758  | 62209  | 2       |
| collateralOwner                                | 681             | 681    | 681    | 681    | 7       |
| increaseDebt                                   | 8547            | 44638  | 62475  | 62475  | 6       |
| increaseDebtAndSell                            | 198559          | 208962 | 198559 | 229769 | 3       |
| isAllowed                                      | 592             | 592    | 592    | 592    | 1       |
| liquidationAuctionMinSpacing                   | 262             | 262    | 262    | 262    | 1       |
| liquidationPenaltyBips                         | 328             | 328    | 328    | 328    | 4       |
| liquidationsLocked                             | 378             | 378    | 378    | 378    | 1       |
| maxDebt                                        | 840             | 3206   | 840    | 25233  | 36      |
| maxLTV                                         | 306             | 306    | 306    | 306    | 2       |
| newTarget                                      | 571             | 571    | 571    | 571    | 1       |
| onERC721Received                               | 40583           | 124829 | 97583  | 267393 | 48      |
| papr                                           | 317             | 317    | 317    | 317    | 68      |
| pool                                           | 461             | 461    | 461    | 461    | 184     |
| purchaseLiquidationAuctionNFT                  | 9430            | 85051  | 89316  | 109994 | 8       |
| reduceDebt                                     | 976             | 4322   | 4845   | 6622   | 4       |
| removeCollateral                               | 7422            | 14848  | 15826  | 26194  | 10      |
| sendPaprFromAuctionFees                        | 2650            | 12718  | 12718  | 22787  | 2       |
| setAllowedCollateral                           | 2801            | 24686  | 25570  | 27570  | 49      |
| setFundingPeriod                               | 2599            | 2599   | 2599   | 2599   | 1       |
| setLiquidationsLocked                          | 2638            | 17330  | 24676  | 24676  | 3       |
| setPool                                        | 2673            | 2673   | 2673   | 2673   | 1       |
| startLiquidationAuction                        | 973             | 47311  | 42708  | 75484  | 21      |
| token0IsUnderlying                             | 268             | 268    | 268    | 268    | 117     |
| uniswapV3SwapCallback                          | 4042            | 53260  | 56431  | 73945  | 25      |
| vaultInfo                                      | 1142            | 1449   | 1142   | 3142   | 26      |

## After:

```solidity
uint256 public constant override liquidationAuctionMinSpacing = 2 days;
```
```solidity
uint256 public constant override perPeriodAuctionDecayWAD = 0.7e18;
```
```solidity
uint256 public constant override auctionDecayPeriod = 1 days;
```
```solidity
uint256 public constant override auctionStartPriceMultiplier = 3;
```
```solidity
uint256 public constant override liquidationPenaltyBips = 1000;
```

| src/PaprController.sol:PaprController contract |                 |        |        |        |         |
|------------------------------------------------|-----------------|--------|--------|--------|---------|
| Deployment Cost                                | Deployment Size |        |        |        |         |
| 10316841                                       | 36108           |        |        |        |         |
| Function Name                                  | min             | avg    | median | max    | # calls |
| addCollateral                                  | 3236            | 66947  | 79852  | 94442  | 26      |
| auctionCurrentPrice                            | 4713            | 6141   | 6713   | 6713   | 7       |
| burnPaprFromAuctionFees                        | 2620            | 3663   | 3663   | 4707   | 2       |
| buyAndReduceDebt                               | 57308           | 59758  | 59758  | 62209  | 2       |
| collateralOwner                                | 681             | 681    | 681    | 681    | 7       |
| increaseDebt                                   | 8547            | 44638  | 62475  | 62475  | 6       |
| increaseDebtAndSell                            | 198559          | 208962 | 198559 | 229769 | 3       |
| isAllowed                                      | 592             | 592    | 592    | 592    | 1       |
| liquidationAuctionMinSpacing                   | 262             | 262    | 262    | 262    | 1       |
| liquidationPenaltyBips                         | 328             | 328    | 328    | 328    | 4       |
| liquidationsLocked                             | 378             | 378    | 378    | 378    | 1       |
| maxDebt                                        | 840             | 3206   | 840    | 25233  | 36      |
| maxLTV                                         | 306             | 306    | 306    | 306    | 2       |
| newTarget                                      | 571             | 571    | 571    | 571    | 1       |
| onERC721Received                               | 40583           | 124829 | 97583  | 267393 | 48      |
| papr                                           | 317             | 317    | 317    | 317    | 68      |
| pool                                           | 461             | 461    | 461    | 461    | 184     |
| purchaseLiquidationAuctionNFT                  | 9430            | 85051  | 89316  | 109994 | 8       |
| reduceDebt                                     | 976             | 4322   | 4845   | 6622   | 4       |
| removeCollateral                               | 7422            | 14848  | 15826  | 26194  | 10      |
| sendPaprFromAuctionFees                        | 2650            | 12718  | 12718  | 22787  | 2       |
| setAllowedCollateral                           | 2801            | 24686  | 25570  | 27570  | 49      |
| setFundingPeriod                               | 2599            | 2599   | 2599   | 2599   | 1       |
| setLiquidationsLocked                          | 2638            | 17330  | 24676  | 24676  | 3       |
| setPool                                        | 2673            | 2673   | 2673   | 2673   | 1       |
| startLiquidationAuction                        | 973             | 47311  | 42708  | 75484  | 21      |
| token0IsUnderlying                             | 268             | 268    | 268    | 268    | 117     |
| uniswapV3SwapCallback                          | 4042            | 53260  | 56431  | 73945  | 25      |
| vaultInfo                                      | 1142            | 1449   | 1142   | 3142   | 26      |