

## [L-01] Unspecific Compiler Version Pragma

Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

```
papr/src/NFTEDA/NFTEDA.sol::2 => pragma solidity >=0.8.0;
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::2 => pragma solidity >=0.8.0;
papr/src/PaprController.sol::2 => pragma solidity ^0.8.17;
papr/src/PaprToken.sol::2 => pragma solidity ^0.8.17;
papr/src/ReservoirOracleUnderwriter.sol::2 => pragma solidity ^0.8.17;
papr/src/UniswapOracleFundingRateController.sol::2 => pragma solidity ^0.8.17;
```

## [L-02] Use of Block.timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

```
papr/src/NFTEDA/NFTEDA.sol::121 => auction.startPrice, block.timestamp - startTime, auction.secondsInPeriod, auction.perPeriodDecayPercentWad
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::44 => auctionState[id] = AuctionState({startTime: uint96(block.timestamp), starter: msg.sender});
papr/src/PaprController.sol::321 => if (block.timestamp - info.latestAuctionStartTime < liquidationAuctionMinSpacing) {
papr/src/PaprController.sol::325 => info.latestAuctionStartTime = uint40(block.timestamp);
papr/src/PaprController.sol::394 => if (_lastUpdated == block.timestamp) {
papr/src/ReservoirOracleUnderwriter.sol::106 => oracleInfo.message.timestamp > block.timestamp || oracleInfo.message.timestamp + VALID_FOR < block.timestamp
papr/src/UniswapOracleFundingRateController.sol::46 => if (_lastUpdated == block.timestamp) {
papr/src/UniswapOracleFundingRateController.sol::55 => _lastUpdated = uint48(block.timestamp);
papr/src/UniswapOracleFundingRateController.sol::64 => if (_lastUpdated == block.timestamp) {
papr/src/UniswapOracleFundingRateController.sol::73 => if (_lastUpdated == block.timestamp) {
papr/src/UniswapOracleFundingRateController.sol::100 => _lastUpdated = uint48(block.timestamp);
papr/src/UniswapOracleFundingRateController.sol::141 => /// @dev reverts if block.timestamp - _lastUpdated == 0
papr/src/UniswapOracleFundingRateController.sol::145 => _lastCumulativeTick, tickCumulative, int56(uint56(block.timestamp - _lastUpdated))
papr/src/UniswapOracleFundingRateController.sol::157 => uint256 period = block.timestamp - _lastUpdated;
```

## [L-03] decimals() not part of ERC20 standard

decimals() is not part of the official ERC20 standard and might fail for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

```
papr/src/PaprController.sol::82 => uint256 underlyingONE = 10 ** underlying.decimals();
```

## [L-04] Unsafe use of transfer()/transferFrom() with IERC20

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert. Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

```
papr/src/PaprController.sol::101 => collateralArr[i].addr.transferFrom(msg.sender, address(this), collateralArr[i].id);
papr/src/PaprController.sol::202 => underlying.transfer(params.swapFeeTo, fee);
papr/src/PaprController.sol::203 => underlying.transfer(proceedsTo, amountOut - fee);
papr/src/PaprController.sol::226 => underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);
papr/src/PaprController.sol::514 => underlying.transfer(params.swapFeeTo, fee);
papr/src/PaprController.sol::515 => underlying.transfer(proceedsTo, amountOut - fee);
papr/src/PaprController.sol::546 => papr.transfer(auction.nftOwner, totalOwed - debtCached);
```

## [L-05] Events not emitted for important state changes

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

```
papr/src/PaprController.sol::350 => function setPool(address _pool) external override onlyOwner {
papr/src/PaprController.sol::355 => function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
papr/src/PaprController.sol::360 => function setLiquidationsLocked(bool locked) external override onlyOwner {
```

## [L-06] _safeMint() should be used rather than _mint() wherever possible

Some tokens do not implement the ERC20 standard properly but are still accepted by most code that accepts ERC20 tokens. For example Tether (USDT)'s transfer() and transferFrom() functions do not return booleans as the specification requires, and instead have no return value. When these sorts of tokens are cast to IERC20, their function signatures do not match and therefore the calls made, revert. Use OpenZeppelin’s SafeERC20's safeTransfer()/safeTransferFrom() instead

```
papr/src/PaprToken.sol::25 => _mint(to, amount);
```

## [N-01] Use a more recent version of solidity

Use a solidity version of at least 0.8.4 to get bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>)
Use a solidity version of at least 0.8.12 to get string.concat() instead of abi.encodePacked(<str>,<str>)
Use a solidity version of at least 0.8.13 to get the ability to use using for with a list of free functions

```
papr/src/NFTEDA/NFTEDA.sol::2 => pragma solidity >=0.8.0;
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::2 => pragma solidity >=0.8.0;
```

## [N-02] Large multiples of ten should use scientific notation

Use (e.g. 1e6) rather than decimal literals (e.g. 1000000), for better code readability

```
papr/src/PaprController.sol::92 => address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);
```

## [N-03] Use scientific notation (e.g. 1e18) rather than exponentiation (e.g. 10**18)

Scientific notation should be used for better code readability

```
papr/src/PaprController.sol::87 => initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);
papr/src/PaprController.sol::89 => initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);
```

## [N-04] Event is missing indexed fields

Each event should use three indexed fields if there are three or more fields

```
papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::21 => event SetAuctionCreatorDiscount(uint256 discount);
```

## [N-05] Missing NatSpec

Code should include NatSpec

```
papr/src/PaprToken.sol::1 => // SPDX-License-Identifier: MIT
```

## [N-06] Missing event for critical parameter change

Emitting events after sensitive changes take place, to facilitate tracking and notify off-chain clients following changes to the contract.

```
papr/src/PaprController.sol::350 => function setPool(address _pool) external override onlyOwner {
papr/src/PaprController.sol::355 => function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
papr/src/PaprController.sol::360 => function setLiquidationsLocked(bool locked) external override onlyOwner {
```

