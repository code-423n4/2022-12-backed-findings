# Gas Optimizations Report for Papr contest
## Overview
During the audit, 2 gas issues were found.

Total savings ~190.
№ | Title | Instance Count | Saved
--- | --- | --- | ---
G-1 | Use unchecked blocks for subtractions where underflow is impossible | 2 | 70
G-2 | Use local variable cache instead of accessing mapping or array multiple times | 3 | 120

## Gas Optimizations Findings(2)
### G-1. Use unchecked blocks for subtractions where underflow is impossible
##### Description
In Solidity 0.8+, there’s a default overflow and underflow check on unsigned integers. When an overflow or underflow isn’t possible (after require or if-statement), some gas can be saved by using [unchecked blocks](https://docs.soliditylang.org/en/v0.8.17/control-structures.html#checked-or-unchecked-arithmetic).
##### Instances
- totalOwed - debtCached: https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L546
- debtCached - totalOwed: https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L550

##### Saved
This saves ~70.
#
### G-2. Use local variable cache instead of accessing mapping or array multiple times
##### Description
It saves gas due to not having to perform the same key’s keccak256 hash and/or offset recalculation.
##### Instances
- cache ```_vaultInfo[auction.nftOwner][auction.auctionAssetContract]```: https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L272
- cache ```_vaultInfo[msg.sender][collateral.addr]```: https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L438
- cache ```_vaultInfo[account][asset]```: https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L465

##### Saved
This saves ~120.