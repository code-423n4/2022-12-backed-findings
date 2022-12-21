## [G-01] `public` function can be changed to `public view`.

The function `underwritePriceForCollateral` in `ReservoirOracleUnderwriter` can be changed to `public view` function to save some gas.

Current implementation:

```solidity
function underwritePriceForCollateral(ERC721 asset, PriceKind priceKind, OracleInfo memory oracleInfo)
        public
        returns (uint256)
    {
```

Recommended:

```solidity
function underwritePriceForCollateral(ERC721 asset, PriceKind priceKind, OracleInfo memory oracleInfo)
        public
				view
        returns (uint256)
    {
```