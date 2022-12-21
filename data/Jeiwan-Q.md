# [L-01] Loss of precision in `oneToOneSqrtRatio`
## Targets
- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L111
## Impact
A Uniswap pool may not be initialized at exact 1:1 price when an underlying token's decimals are not 18.
## Proof of Concept
When a PaprController is deployed and initialized it deploys a Uniswap V3 pool with Papr and an underlying token ([PaprController.sol#L92](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L92)). The pool will serve as the market price discovery mechanism of Papr: changes in price of Papr in the pool will affect the interest rate of Papr. Initially, the price of Papr is set to 1:1 with the underlying token ([PaprController.sol#L85-L90](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L85-L90)), however the calculation of such a price is not precise: the numerator is shifted left by 96 bits, instead of 192, which reduces the precision of the square root operation ([UniswapHelpers.sol#L110-L112](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L110-L112)):
```solidity
function oneToOneSqrtRatio(uint256 token0ONE, uint256 token1ONE) internal pure returns (uint160) {
    return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
}
```

For example, when calling this function with `token0ONE = 1e18` and `token1ONE = 1e6`, we get `79228267247129223624114`, which is `(79228267247129223624114 / 2**96) ** 2 = 1.0000026438309507e-12` and `1 / 1.0000026438309507e-12 = 999997356176.0391`, which is not precise 1:1.

Here are more examples:
```
  Error: 18 and 6
  Error: a == b not satisfied [uint]
    Expected: 79228162514264337593543
      Actual: 79228267247129223624114
  Error: 6 and 18
  Error: a == b not satisfied [uint]
    Expected: 79228162514264337593543950336000000
      Actual: 79228057781537899283318961129827820
  Error: 18 and 8
  Error: a == b not satisfied [uint]
    Expected: 792281625142643375935439
      Actual: 792282497916421281862280
  Error: 8 and 18
  Error: a == b not satisfied [uint]
    Expected: 7922816251426433759354395033600000
      Actual: 7922807523698269125109939133199873
```

```solidity
// test/UniswapHelpers.t.sol

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.17;

import "forge-std/Test.sol";

import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";
import "../src/libraries/UniswapHelpers.sol";

contract UniswapHelpersTest is Test {
  function testOneToOneSqrtRatio_AUDIT() public {
    assertEq(
      UniswapHelpers.oneToOneSqrtRatio(1e18, 1e18),
      oneToOneSqrtRatio(1e18, 1e18),
      "18 and 18"
    );

    assertEq(
      UniswapHelpers.oneToOneSqrtRatio(1e18, 1e6),
      oneToOneSqrtRatio(1e18, 1e6),
      "18 and 6"
    );

    assertEq(
      UniswapHelpers.oneToOneSqrtRatio(1e6, 1e18),
      oneToOneSqrtRatio(1e6, 1e18),
      "6 and 18"
    );

    assertEq(
      UniswapHelpers.oneToOneSqrtRatio(1e18, 1e8),
      oneToOneSqrtRatio(1e18, 1e8),
      "18 and 8"
    );

    assertEq(
      UniswapHelpers.oneToOneSqrtRatio(1e8, 1e18),
      oneToOneSqrtRatio(1e8, 1e18),
      "8 and 18"
    );
  }

  function oneToOneSqrtRatio(uint256 token0ONE, uint256 token1ONE) internal pure returns (uint160) {
    return uint160(FixedPointMathLib.sqrt((token1ONE << 192) / token0ONE));
  }
}
```

## Tools Used
Manual review
## Recommended Mitigation Steps
Consider this change:
```diff
--- a/src/libraries/UniswapHelpers.sol
+++ b/src/libraries/UniswapHelpers.sol
@@ -6,6 +6,7 @@ import {IUniswapV3Factory} from "v3-core/contracts/interfaces/IUniswapV3Factory.
 import {TickMath} from "fullrange/libraries/TickMath.sol";
 import {FullMath} from "fullrange/libraries/FullMath.sol";
 import {SafeCast} from "v3-core/contracts/libraries/SafeCast.sol";
+import {FixedPointMathLib} from "solmate/utils/FixedPointMathLib.sol";

 import {PoolAddress} from "./PoolAddress.sol";

@@ -108,6 +109,6 @@ library UniswapHelpers {
     /// @param token1ONE 10 ** token1.decimals()
     /// @return sqrtRatio at which token0 and token1 are trading at 1:1
     function oneToOneSqrtRatio(uint256 token0ONE, uint256 token1ONE) internal pure returns (uint160) {
-        return TickMath.getSqrtRatioAtTick(TickMath.getTickAtSqrtRatio(uint160((token1ONE << 96) / token0ONE)) / 2);
+        return uint160(FixedPointMathLib.sqrt((token1ONE << 192) / token0ONE));
     }
 }
```



# [L-02] Auction fees cannot be withdrawn since they're always burned
## Targets
- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L382-L384
- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L540
## Impact
The `sendPaprFromAuctionFees` and `burnPaprFromAuctionFees` functions of `PaprController` won't ever be used because the contract doesn't accumulate Papr tokens: excess penalties are always burned during auction.
## Proof of Concept
`PaprController` implements `sendPaprFromAuctionFees` and `burnPaprFromAuctionFees` ([PaprController.sol#L382-L388](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L382-L388)) which intended to send or burn Papr tokens collected as auction fees. However, the auction doesn't collect fees ([PaprController.sol#L264-L345](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L264-L345)). It does apply a penalty to the excess amount resulted after selling of an NFT, but the penalty amount is burned immediately ([PaprController.sol#L536-L540](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L536-L540)):
```solidity
uint256 fee = excess * liquidationPenaltyBips / BIPS_ONE;
uint256 credit = excess - fee;
uint256 totalOwed = credit + neededToSaveVault;

PaprToken(address(papr)).burn(address(this), fee);
```

The following test demonstrates that no Papr tokens are accumulated by `PaprController` during a liquidation:
```solidity
// test/paprController/PurchaseLiquidationAuctionNFT.t.sol

function testAuctionFeesAreBurned_AUDIT() public {
    // add collateral
    uint256 tokenId = collateralId + 5;
    nft.mint(borrower, tokenId);
    vm.stopPrank();
    vm.startPrank(borrower);
    nft.approve(address(controller), tokenId);
    collateral.id = tokenId;
    IPaprController.Collateral[] memory c = new IPaprController.Collateral[](1);
    c[0] = collateral;
    controller.addCollateral(c);
    vm.stopPrank();
    vm.startPrank(purchaser);

    /// https://www.wolframalpha.com/input?i=solve+4+%3D+8.999+*+0.3+%5E+%28x+%2F+86400%29
    vm.warp(block.timestamp + 58187);
    oracleInfo = _getOracleInfoForCollateral(collateral.addr, underlying);
    IPaprController.VaultInfo memory info = controller.vaultInfo(borrower, collateral.addr);

    // Calculating auction penalty from the excess amount.
    uint256 neededToSave = info.debt - controller.maxDebt(oraclePrice);
    uint256 excess = controller.auctionCurrentPrice(auction) - neededToSave;
    uint256 penalty = excess * controller.liquidationPenaltyBips() / 1e4;

    controller.papr().approve(address(controller), auction.startPrice);

    // Expecting the penalty to be burned.
    vm.expectEmit(true, true, false, true);
    emit Transfer(address(controller), address(0), penalty);
    controller.purchaseLiquidationAuctionNFT(auction, auction.startPrice, purchaser, oracleInfo);

    // Papr balance of the controller is 0.
    assertEq(controller.papr().balanceOf(address(controller)), 0);
}
```

## Tools Used
Manual review
## Recommended Mitigation Steps
Consider removing or renaming the `sendPaprFromAuctionFees` and `burnPaprFromAuctionFees` functions.