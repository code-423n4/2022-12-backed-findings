# Gas Optimizations

## Summary

|               | Issue         | Instances     |
| :-------------: |:-------------|:-------------:|
| 1   | `auctionCreatorDiscountPercentWad` and `_pricePercentAfterDiscount` state variables should be declared `immutable` | 2 |
| 2   | Variables inside `struct` should be packed to save gas  | 2  |
| 3   | The function `purchaseLiquidationAuctionNFT` can be optimized to save gas |  1 |
| 4   | Use `unchecked` blocks to save gas  |  4 |
| 5   | `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables  |  2 |


## Findings

## 1- `auctionCreatorDiscountPercentWad` and `_pricePercentAfterDiscount` state variables should be declared `immutable` :

The state variables `auctionCreatorDiscountPercentWad` and `_pricePercentAfterDiscount` are only set in the constructor of `NFTEDAStarterIncentive` and their values can't be changed later on (there is no external/public function in PaprController contract to update them), so they should be marked `immutable` this will avoid a Gsset (20000 gas) in the constructor and replaces each Gwarmacces (100 gas) with a PUSH32 (3 gas).

There are 2 instances of this issue:

File: src/NFTEDA/extensions/NFTEDAStarterIncentive.sol [Line 26-27](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L26-L27)
```
uint256 public auctionCreatorDiscountPercentWad;
uint256 internal _pricePercentAfterDiscount;
```

## 2- Variables inside `struct` should be packed to save gas :

As the solidity EVM works with 32 bytes, variables less than 32 bytes should be packed inside a struct so that they can be stored in the same slot, each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

There are 2 instances of this issue:

File: src/NFTEDA/interfaces/INFTEDA.sol [Line 10-26](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/interfaces/INFTEDA.sol#L10-L26)

```
struct Auction {
    // the nft owner
    address nftOwner;
    // the nft token id
    uint256 auctionAssetID;
    // the nft contract address
    ERC721 auctionAssetContract;
    // How much the auction price will decay in each period
    // expressed as percent scaled by 1e18, i.e. 1e18 = 100%
    uint256 perPeriodDecayPercentWad;
    // the number of seconds in the period over which perPeriodDecay occurs
    uint256 secondsInPeriod;
    // the auction start price
    uint256 startPrice;
    // the payment asset and quote asset for startPrice
    ERC20 paymentAsset;
}
```

As the values of `perPeriodDecayPercentWad` and `secondsInPeriod` can't really overflow 2^128 (the first has a max value of 1e18 and the second is a period in seconds which can not realistically overflow) their values should be packed together and the struct should be modified as follow to save gas :

```
struct Auction {
    // the nft owner
    address nftOwner;
    // the nft token id
    uint256 auctionAssetID;
    // the nft contract address
    ERC721 auctionAssetContract;
    // How much the auction price will decay in each period
    // expressed as percent scaled by 1e18, i.e. 1e18 = 100%
    uint128 perPeriodDecayPercentWad;
    // the number of seconds in the period over which perPeriodDecay occurs
    uint128 secondsInPeriod;
    // the auction start price
    uint256 startPrice;
    // the payment asset and quote asset for startPrice
    ERC20 paymentAsset;
}
```


File: src/interfaces/IPaprController.sol [Line 31-42](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L31-L42)

```
struct SwapParams {
    /// @dev amount of input token to swap
    uint256 amount;
    /// @dev minimum amount of output token to be received
    uint256 minOut;
    /// @dev sqrt price limit for the swap
    uint160 sqrtPriceLimitX96;
    /// @dev optional address to receive swap fees
    address swapFeeTo;
    /// @dev optional swap fee in bips
    uint256 swapFeeBips;
}
```

As the value of `swapFeeBips` can't really overflow 2^96 as it has a max value of 1e18, so it should be packed with `sqrtPriceLimitX96` to save gas and the struct should be modified as follow :

```
struct SwapParams {
    /// @dev amount of input token to swap
    uint256 amount;
    /// @dev minimum amount of output token to be received
    uint256 minOut;
    /// @dev sqrt price limit for the swap
    uint160 sqrtPriceLimitX96;
    /// @dev optional swap fee in bips
    uint96 swapFeeBips;
    /// @dev optional address to receive swap fees
    address swapFeeTo;
}
```

### 3- The function `purchaseLiquidationAuctionNFT` can be optimized to save gas :

In the `purchaseLiquidationAuctionNFT` function there is the follwing lines of code which are used to calculate `collateralValueCached` :

File: src/PaprController.sol [Line 270-273](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L270-L273)
```
uint256 collateralValueCached = underwritePriceForCollateral(
    auction.auctionAssetContract, ReservoirOracleUnderwriter.PriceKind.TWAP, oracleInfo
) * _vaultInfo[auction.nftOwner][auction.auctionAssetContract].count;
bool isLastCollateral = collateralValueCached == 0;
```

Those lines can be rearranged as follow to save gas at each call :

```
bool isLastCollateral; // by default set to false
uint256 collateralValueCached; // by default set to 0

uint16 count = _vaultInfo[auction.nftOwner][auction.auctionAssetContract].count;

if (count == 0){
    isLastCollateral = true;
    // no need to calculate collateralValueCached as it will remain equal to 0
} else {
    collateralValueCached = underwritePriceForCollateral(
        auction.auctionAssetContract, ReservoirOracleUnderwriter.PriceKind.TWAP, oracleInfo
    ) * count;
}
```

In this case the function will not call the `underwritePriceForCollateral` function when count is zero which will cost less gas for the sender.


### 4- Use `unchecked` blocks to save gas :

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

There are 4 instances of this issue:

1-  File: src/PaprController.sol [Line 278](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L278)
```
uint256 neededToSaveVault = maxDebtCached > debtCached ? 0 : debtCached - maxDebtCached;
```

The operation `debtCached - maxDebtCached` cannot underflow because of the check `maxDebtCached > debtCached` which came before it, so the operation should be marked unchecked.


2-  File: src/PaprController.sol [Line 280](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L280)
```
uint256 excess = price > neededToSaveVault ? price - neededToSaveVault : 0;
```

The operation `price - neededToSaveVault` cannot underflow because of the check `price > neededToSaveVault` which came before it, so the operation should be marked unchecked.


3-  File: src/PaprController.sol [Line 537](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L537)
```
uint256 credit = excess - fee;
```

The above operation cannot underflow because of the line code which ensures that we always have `fee <= excess` : 

File: src/PaprController.sol [Line 536](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L536)
```
uint256 fee = excess * liquidationPenaltyBips / BIPS_ONE;
```

So the operation should be marked unchecked.


4-  File: src/PaprController.sol [Line 550](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L550)
```
remaining = debtCached - totalOwed;
```

The above operation cannot underflow because of the check : 

File: src/PaprController.sol [Line 542](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L542)
```
if (totalOwed > debtCached) 
```

So the operation should be marked unchecked.


### 5- `x += y/x -= y` costs more gas than `x = x + y/x = x - y` for state variables :

Using the addition operator instead of plus-equals saves **113 gas** as explained [here](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)

There are 2 instances of this issue:

File: src/PaprController.sol [Line 326](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326)
```
info.count -= 1;
```

File: src/PaprController.sol [Line 419](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419)
```
_vaultInfo[account][collateral.addr].count += 1;
```