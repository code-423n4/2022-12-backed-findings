# QA Report

## 1. CryptoPunk Not Supported
In the whitepaper it takes PUNKs as an example; However, since cryptopunks NFT does not compatible with the ERC721 standard, this kind of asset cannot be collateralized or auctioned here.

## 2. src/PaprController.sol
Immutable variables like `liquidationAuctionMinSpacing` should be initialized in constructor rather than hard code them; Some variables like `liquidationPenaltyBips` should NOT be set as immutable, maybe should be changeable by the owner and later moved to DAO.

## 3. src/PaprController.sol#L109
In the `removeCollateral` function, it only supports remove collateral of the same NFT collection in one call. This check is unnecessary, since we could query oracle for each address and store them temporarily, or create some internal function behaves like `updateTarget`, just query oracle when `lastupdated` of this address is not `block.timestamp`.