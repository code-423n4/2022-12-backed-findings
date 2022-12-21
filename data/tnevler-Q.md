# Report
## Non-Critical Issues ##
### [N-1]: Function defines a named return variable but then it uses return statements
**Context:**

```return _target;``` [L47](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L47) 

**Recommendation:**

Choose named return variable or return statement. It is unnecessary to use both.

### [N-2]: Variable is unused
**Context:**

```function _auctionCurrentPrice(uint256 id, uint256 startTime, INFTEDA.Auction memory auction)``` [L114](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L114) (id)

### [N-3]: Wrong order of functions
**Context:**

```function lastUpdated() external view override returns (uint256) {``` [L81](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L81) (external function can not go after public function)

**Description:**

According to [official solidity documentation](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions) functions should be grouped according to their visibility and ordered:

+ constructor

+ receive function (if exists)

+ fallback function (if exists)

+ external

+ public

+ internal

+ private

Within a grouping, place the view and pure functions last.

**Recommendation:**

Put the functions in the correct order according to the documentation.

### [N-4]: NatSpec is missing
**Context:**

1. ```contract PaprToken is ERC20 {``` [L6](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L6) delete
1. ```function uniswapV3SwapCallback(int256 amount0Delta, int256 amount1Delta, bytes calldata _data) external {``` [L234](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L234) 
1. ```function burnPaprFromAuctionFees(uint256 amount) external override onlyOwner {``` [L386](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L386) 
1. ```function _addCollateralToVault(address account, IPaprController.Collateral memory collateral) internal {``` [L413](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L413) 
1. ```function _removeCollateral(``` [L424](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L424) 
1. ```function _increaseDebt(``` [L456](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L456) 
1. ```function _reduceDebt(address account, ERC721 asset, address burnFrom, uint256 amount) internal {``` [L481](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L481) 
1. ```function _reduceDebtWithoutBurn(address account, ERC721 asset, uint256 amount) internal {``` [L486](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L486) 
1. ```function _increaseDebtAndSell(``` [L493](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L493) 
1. ```function _purchaseNFTAndUpdateVaultIfNeeded(Auction calldata auction, uint256 maxPrice, address sendTo)``` [L519](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L519) 
1. ```function _handleExcess(uint256 excess, uint256 neededToSaveVault, uint256 debtCached, Auction calldata auction)``` [L532](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L532) 
1. ```function _maxDebt(uint256 totalCollateraValue, uint256 cachedTarget) internal view returns (uint256) {``` [L556](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L556) 
1. ```library EDAPrice {``` [L7](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/libraries/EDAPrice.sol#L7) delete
1. ```library OracleLibrary {``` [L8](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L8) delete

### [N-5]: Typos
**Context:**

1. ```/// @return selector indicating succesful receiving of the NFT``` [L158](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L158) (Change ***succesful*** to ***successful***)
1. ```/// @param auction The defintion of the auction``` [L46](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L46) (Change ***defintion*** to ***definition***)
1. ```/// @dev Derived from the auction. Identitical auctions cannot exist simultaneously``` [L58](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/interfaces/INFTEDA.sol#L58) (Change ***Identitical*** to ***Identical***)
1. ```/// @notice if liquidations are currently locked, meaning startLiquidationAuciton will revert``` [L237](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L237) (Change ***startLiquidationAuciton*** to ***startLiquidationAuction***)

### [N-6]: Line is too long
**Context:**

1. ```/// @dev vaults are uniquely identified by the address of the vault owner and the address of the collateral token used in the vault``` [L66](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L66) 
1. ```/// @notice removes debt from a vault and burns it by buying it on Uniswap in exchange for the controller's underlying token``` [L164](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L164) 
1. ```/// @notice the multiplier for the starting price of an auction, applied to the current price of the collateral in papr tokens``` [L257](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L257) 
1. ```/// @notice fee paid by the vault owner when their vault is liquidated if there was excess debt credited to their vault, in bips``` [L260](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L260) 

**Description:**

Maximum suggested line length is [120 characters](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#maximum-line-length).
