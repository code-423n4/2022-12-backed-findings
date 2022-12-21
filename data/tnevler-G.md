# Report
## Gas Optimizations ##
### [G-1]: Place subtractions where the operands can't underflow in unchecked {} block
**Context:**

1. ```papr.transfer(auction.nftOwner, totalOwed - debtCached);``` [L546](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L546) (totalOwed - debtCached)
1. ```remaining = debtCached - totalOwed;``` [L550](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L550) (debtCached - totalOwed)

**Description:**

Some gas can be saved by using an unchecked {} block if an underflow isn't possible because of a previous require() or if-statement.

### [G-2]: Cache a value from a mapping/array in local memory
**Context:**

1. ```) * _vaultInfo[auction.nftOwner][auction.auctionAssetContract].count;``` [L272](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L272) (_vaultInfo[auction.nftOwner][auction.auctionAssetContract])
1. ```newCount = _vaultInfo[msg.sender][collateral.addr].count - 1;``` [L438](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L438) (_vaultInfo[msg.sender][collateral.addr] and in L449 use it)
1. ```uint256 newDebt = _vaultInfo[account][asset].debt + amount;``` [L465](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L465) (_vaultInfo[account][asset] also used in L469)

**Description:**

If you read value from mapping/array more than once within a function then it is cheaper to cache it in local memory and then read it from memory wnen it is neaded. This will save about 40 gas.

