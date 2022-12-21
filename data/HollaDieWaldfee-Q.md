# Papr Low Risk and Non-Critical Issues
## Summary
| Risk      | Title | File | Instances
| ----------- | ----------- | ----------- | ----------- |
| L-00      | Check that `targetMarkRatioMax > targetMarkRatioMin` | UniswapOracleFundingRateController.sol | 1 |
| L-01      | Liquidation Fees are burnt and cannot be withdrawn by admin | PaprController.sol | 1 |
| L-02      | `buyAndReduceDebt` function reverts if swap fees are used | PaprController.sol | 1 |
| N-00      | Function state mutability can be restricted to pure | - | 2 |
| N-01      | Function state mutability can be restricted to view | - | 1 |
| N-02      | `onERC721Received` function: check `amount > 0` instead of `minOut > 0` | PaprController.sol | 1 |

## [L-00] Check that `targetMarkRatioMax > targetMarkRatioMin`
[https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L38-L39](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L38-L39)  

In the constructor of `UniswapOracleFundingRateController` it should be checked that `targetMarkRatioMax > targetMarkRatioMin` as otherwise the values do not make sense and cause issues in the calculation of the funding rate.  

## [L-01] Liquidation Fees are burnt and cannot be withdrawn by admin
The Whitepaper ([https://backed.mirror.xyz/8SslPvU8of0h-fxoo6AybCpm51f30nd0qxPST8ep08c](https://backed.mirror.xyz/8SslPvU8of0h-fxoo6AybCpm51f30nd0qxPST8ep08c)) states that:  

> If there is excess papr, a liquidation fee is subtracted and given to the protocol (added to an insurance fund) and the remainder is used to further reduce the borrowers debt  

In order to make use of the Liquidation fees and to transfer them to the insurance fund, the PaprController has the `sendPaprFromAuctionFees` function ([https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L382-L384](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L382-L384)).  

However the liquidation fees are instantly burnt once they are calculated after the NFT purchase:  
[https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L540](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L540)  

So there are no fees to be withdrawn.  

In order to fix this, just remove the above line that burns the fees.  

## [L-02] `buyAndReduceDebt` function reverts if swap fees are used
The `PaprController.buyAndReduceDebt` function mistakenly sends the swap fees (which is an optional feature the msg.sender may or may not use) from the PaprController contract to the `swapFeeTo` address:  
[https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L225-L227](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L225-L227)  

This is wrong. The swap fees should be transferred from `msg.sender` to the `swapFeeTo` address.  

The PaprController does not hold any `underlying` tokens and so if swap fees are supposed to be used, the function call will revert, making the optional swap fee feature unusable.  

To fix this, change the line that transfers the tokens to:  
```solidity
underlying.transferFrom(msg.sender, params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);
```

## [N-00] Function state mutability can be restricted to pure
In Solidity, functions should make use of the most restrictive state mutability modifier possible.  

The following functions can be `pure`:  

[https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/libraries/EDAPrice.sol#L10-L23](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/libraries/EDAPrice.sol#L10-L23)  

[https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L34-L53](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L34-L53)  


## [N-01] Function state mutability can be restricted to view
In Solidity, functions should make use of the most restrictive state mutability modifier possible.  

The following functions can be `view`:  

[https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L64-L117](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L64-L117)  

## [N-03] `onERC721Received` function: check `amount > 0` instead of `minOut > 0`
The `PaprController.onERC721Received` function allows to optionally call `_increaseDebtAndSell` or `_increaseDebt`.  
`_increaseDebtAndSell` is called when `request.swapParams.minOut > 0` ([https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L170](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L170)).  

Instead of `request.swapParams.minOut > 0` it should be checked that `request.swapParams.amount > 0`.  
This is because `minOut` should be an optional parameter that can be left at zero whereas `amount` is a required parameter for the swap.  
