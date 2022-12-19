Papr Gas Report
===

### Cache storage pointers
Storage locations which need to be computed and then read from or written to multiple times, such as reading from a `mapping`, should be cached as a `storage` pointer.
| File                    | Lines                         | Description |
|---|---|---|
| `src/PaprController.sol` | 272, 275 | `vaultInfo[auction.nftOwner][auction.auctionAssetContract]` |
| `src/PaprController.sol` | 311, 330 | `collateralOwner[collateral.addr][collateral.id]` |
| `src/PaprController.sol` | 438, 439, 446 | `_vaultInfo[msg.sender][collateral.addr]` |
| `src/PaprController.sol` | 465, 469, 475 | `_vaultInfo[account][asset]` |
| `src/PaprController.sol` | 487 | `_vaultInfo[account][asset]` |
| `src/PaprController.sol` | 525, 526 | `_vaultInfo[auction.nftOwner][auction.auctionAssetContract]` |