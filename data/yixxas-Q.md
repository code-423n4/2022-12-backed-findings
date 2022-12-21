## L-01 - Current decay percentage could be too high

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L44-L47

Currently, we have a decay per period of 70% and a period of 1 day. This means that every day, price will drop be 70%. While I understand that the protocol strives for an exponential decay dutch auction format, with the current numbers, price of NFT will quickly be negligible.

An example of how quickly the price can drop.

Day 0: 1000
Day 1: 300
Day 2: 90
Day 3: 27

## Recommendation
One recommendation is to reduce this amount to a more reasonable 30-50%. It still maintains the property of exponential decrease, but at a slower pace.

&nbsp;

## L02 - `latestAuctionStartTime` can be wrongly set to 0 even if an NFT is still selling in auction

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L519-L530

This issue can happen when multiple collaterals are sent for auction. The vulnerability can happens due to `_purchaseNFTAndUpdateVaultIfNeeded()`.

[PaprController.sol#L519-L530](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L519-L530)
```solidity
function _purchaseNFTAndUpdateVaultIfNeeded(Auction calldata auction, uint256 maxPrice, address sendTo)
    internal
    returns (uint256)
{
    (uint256 startTime, uint256 price) = _purchaseNFT(auction, maxPrice, sendTo);

    if (startTime == _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime) {
        _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime = 0;
    }   

    return price;
}
```

We note how `_vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime` is set to 0 when `startTime == _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime`.

It is documented that when `latestAuctionStartTime == 0`, no auction is being held but this is not true.

2 collaterals from a user can be sent to an auction, but the later one gets purchased first. This would set the `vault.latestAuctionStartTime` to 0 even though the first auction is still running. This can lead to potential problems in the future if we rely on this value.

## Recommendation
It might be a good idea to restrict only one collateral to be sent to the auction at a time. Another high severity issue arises due to this as I have wrote in my other report.

&nbsp;

## L-03 - Using the 30 days TWAP floor price of the entire collection means that the protocol is largely restricted to using the NFTS that are close to the floor price.

There is currently a huge difference in price between the top few PUNK NFT, and the bottom few. For instance, the lowest ask price is currently `63.66 ETH` ( as of the time of writing this report ) as seen [here](https://www.larvalabs.com/cryptopunks). The top few NFTS are last sold in the range of 1000s of ETH. 

Since the value of the debt that a user can raise from the collateral is computed by the total number of deposited collaterals multiplied by the lowest price of the NFT in the collection, it makes little sense of anyone to use any higher-valued NFTs as collateral. This seriously limits the use of the protocol if only a limited number of NFTS are used.

## Recommendation
It is recommended that we use a different metric to measure price here. We could target NFTs indivudally instead of seeing them as a group.

&nbsp;

## L-04 - Signature scheme is not checking that `signerAddress` is not 0

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L88

`ecrecover` returns a value of 0 for invalid signature. We also note that in the constructor, there is no check to ensure that `oracleSigner` is not address 0.

The only check for validity of signature is this,

```solidity
if (signerAddress != oracleSigner) {
    revert IncorrectOracleSigner();
}
```

If `oracleSigner` is set to `address(0)` then a malicious user can pass any price it wants into `oracleInfo` to bypass the check of signature used in `underwritePriceForCollateral()` and hence liquidate any collateral of any user, as well as being able to purchase this liquidated NFT at a price of 0.

## Recommendation
It is recommendated to add the check `if(signerAddress == address(0)) revert Error()`.

&nbsp;

## L-05 - Using only the lowest price of the NFT of the entire collection can be dangerous

Liquidation is decided based on the 30 days TWAP of the floor price of the collection. This might be manipulatable as we only have to manipulate a single NFT to drastically decrease the maximum debt that a user can hold since calculation is done by

> `Total number of NFTS * floor price`

An attacker can possibly control the price of a single NFT, and liquidate many users.

&nbsp;

## R-01 - More accurate to use <= for validity of oracle timestamp

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L106

Currently, the check for oracle timestamp to not exceed `block.timestamp` is done with a strict comparison. Using `<=` can be better here as `VALID_FOR` implies that the oracle timestamp would be valid for the entire duration.

> `oracleInfo.message.timestamp + VALID_FOR < block.timestamp`

## Recommendation
Change to ``oracleInfo.message.timestamp + VALID_FOR <= block.timestamp`

