# Report

## QA Report (Non Critical and Low level Issue Findings)

| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | NATSPEC IS MISSING | 1 |
| [NC-2](#NC-2) | SIGNATURE MALLEABILITY OF EVM’S ECRECOVER() | 1 |
| [NC-3](#NC-3) | LINES ARE TOO LONG | 1 |
| [NC-4](#NC-4) | LOSS OF PRECISION | 6 |
| [NC-5](#NC-5) | PARAMETER NAME IS SAME AS FUNCTION NAME | 1 |
| [NC-6](#NC-6) | UNUSED PARAMETER CAN BE REMOVED | 1 |
| [NC-7](#NC-7) | FUNCTIONS SHOULD NOT RETURN ANY VALUE IF IT IS NOT UTILISED AT CALLING | 2 |
| [L-1](#L-1) | IT IS RECOMMENDED TO USE VIEW FOR FUNCTIONS WHICH DOESN'T CHANGE STATE | 2 |
| [L-2](#L-2) | IT IS RECOMMENDED TO USE PURE FUNCTIONS WHENEVER POSSIBLE IN PLACE OF VIEW | 2 |
| [L-3](#L-3) | UNSAFE ERC721 OPERATIONS | 2 |
| [L-4](#L-4) | MORE UNSAFE ERC20 OPERATIONS | 1 |


###  [NC-1] NATSPEC IS MISSING 
NatSpec is missing for the following function:

*Instances (1)*:
```solidity
File: src/PaprController.sol

234:             function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata _data) external {

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L234)

###  [NC-2] SIGNATURE MALLEABILITY OF EVM’S ECRECOVER() 
The function calls the Solidity ecrecover() function directly to verify the given signatures. However, the ecrecover() EVM opcode allows malleable (non-unique) signatures and thus is susceptible to replay attacks.

Although a replay attack seems not possible for this contract, I recommend using the battle-tested OpenZeppelin’s ECDSA library.

Use the ecrecover function from OpenZeppelin’s ECDSA library for signature verification. (Ensure using a version > 4.7.3 for there was a critical bug >= 4.1.0 < 4.7.3).

*Instances (1)*:
```solidity
File: src/ReservoirOracleUnderwriter.sol

68:             address signerAddress = ecrecover(

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L68)

###  [NC-3] LINES ARE TOO LONG
Usually lines in source code are limited to 80 characters. Today’s screens are much larger so it’s reasonable to stretch this in some cases. Since the files will most likely reside in GitHub, and GitHub starts using a scroll bar in all cases when the length is over 164 characters, the line below should be split when they reach that length Reference: https://docs.soliditylang.org/en/v0.8.10/style-guide.html#maximum-line-length

*Instances (1)*:
```solidity
File: src/ReservoirOracleUnderwriter.sol

111:             (address oracleQuoteCurrency, uint256 oraclePrice) = abi.decode(oracleInfo.message.payload, (address, uint256));

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L111)

###  [NC-4] LOSS OF PRECISION

*Instances (6)*:
```solidity
File: src/PaprController.sol

201:             uint256 fee = amountOut * params.swapFeeBips / BIPS_ONE;
226:             underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);
341:             startPrice: (oraclePrice * auctionStartPriceMultiplier) * FixedPointMathLib.WAD / cachedTarget,
513:             uint256 fee = amountOut * params.swapFeeBips / BIPS_ONE;
536:             uint256 fee = excess * liquidationPenaltyBips / BIPS_ONE;
558:             return maxLoanUnderlying / cachedTarget;

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#201)

###  [NC-5] PARAMETER NAME IS SAME AS FUNCTION NAME
It is Recommended to Use unique name for parameter Variable.
Here, target is used as parameter Variable even when there exists a Function named target inside same file.
*Instances (1)*:
```solidity
File: src/UniswapOracleFundingRateController.sol

95:             function _init(uint256 target, address _pool) internal {

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#95)

###  [NC-6] UNUSED PARAMETER CAN BE REMOVED 
Here the id Parameter is never Used inside the Function.
*Instances (1)*:
```solidity
File: src/NFTEDA/NFTEDA.sol

114:             function _auctionCurrentPrice(uint256 id, uint256 startTime, INFTEDA.Auction memory auction)

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#114)

###  [NC-7] FUNCTIONS SHOULD NOT RETURN ANY VALUE IF IT IS NOT UTILISED AT CALLING
Go to the Function Definition of the Following Functions and Remove the Returns Part.
*Instances (2)*:
```solidity
File: src/PaprController.sol

171:              _increaseDebtAndSell(from, request.proceedsTo, ERC721(msg.sender), request.swapParams, request.oracleInfo);

332:              _startAuction(


```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#171)

###  [L-1] IT IS RECOMMENDED TO USE VIEW FOR FUNCTIONS WHICH DOESN'T CHANGE STATE
It will prevent you from accidentally writing contract state in functions where you don't want to.

*Instances (2)*:
```solidity
File: src/ReservoirOracleUnderwriter.sol

64:             function underwritePriceForCollateral(ERC721 asset, PriceKind priceKind, OracleInfo memory oracleInfo)

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#64)

```solidity
File: src/libraries/UniswapHelpers.sol

82:             function poolCurrentTick(address pool) internal returns (int24) {

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/Libraries/UniswapHelpers.sol#82)

###  [L-2] IT IS RECOMMENDED TO USE PURE FUNCTIONS WHENEVER POSSIBLE IN PLACE OF VIEW

*Instances (2)*:
```solidity
File: src/NFTEDA/libraries/EDAPrice.sol

15:             ) internal pure returns (uint256) {

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/libraries/EDAPrice.sol#15)

```solidity
File: src/libraries/OracleLibrary.sol

34:             function timeWeightedAverageTick(int56 startTick, int56 endTick, int56 twapDuration)
35:             internal
36:             view

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/Libraries/OracleLibrary.sol#36)


###  [L-3] UNSAFE ERC721 OPERATIONS

*Instances (2)*:
```solidity
File: src/NFTEDA/NFTEDA.sol

91:             auction.auctionAssetContract.safeTransferFrom(address(this), sendTo, auction.auctionAssetID);

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#91)

```solidity
File: src/PaprController.sol

444:             collateral.addr.safeTransferFrom(address(this), sendTo, collateral.id);

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#91)

###  [L-4] MORE UNSAFE ERC20 OPERATIONS

*Instances (2)*:
```solidity
File: src/NFTEDA/NFTEDA.sol

91:             auction.auctionAssetContract.safeTransferFrom(address(this), sendTo, auction.auctionAssetID);

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#91)

```solidity
File: src/PaprController.sol

444:             collateral.addr.safeTransferFrom(address(this), sendTo, collateral.id);

```
[Link to code](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#91)
