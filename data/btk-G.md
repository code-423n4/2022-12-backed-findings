## G-1: immutable variables that aren't declared in the constructor should be declared constant instead

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L41-L54

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

| src/PaprController.sol:PaprController contract |                 |
|------------------------------------------------|-----------------|
| Deployment Cost                                | Deployment Size |
| 10374332                                       | 36526           |


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

| src/PaprController.sol:PaprController contract |                 |
|------------------------------------------------|-----------------|
| Deployment Cost                                | Deployment Size |
| 10316841                                       | 36108           |

## G-2: ++ and -- cost less then += 1 and -= 1 

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419
- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326

## Before:

```solidity
info.count -= 1;
```
```solidity
_vaultInfo[account][collateral.addr].count += 1;
```
| src/PaprController.sol:PaprController contract |                 |
|------------------------------------------------|-----------------|
| Deployment Cost                                | Deployment Size |
| 10374332                                       | 36526           |


## After:

```solidity
info.count--;
```
```solidity
_vaultInfo[account][collateral.addr].count++;
```
| src/PaprController.sol:PaprController contract |                 |
|------------------------------------------------|-----------------|
| Deployment Cost                                | Deployment Size |
| 10374132                                       | 36525           |