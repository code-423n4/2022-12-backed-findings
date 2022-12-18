### 1 Instead of immutable change it to constant because you are not initializing the values in the constructor:
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L41-L54


### Unsafe ERC20 Operation(s)

#### Impact
Issue Information: [L001](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l001---unsafe-erc20-operations)

#### Findings:
```
examples/src/PaprController.sol::101 => collateralArr[i].addr.transferFrom(msg.sender, address(this), collateralArr[i].id);
examples/src/PaprController.sol::202 => underlying.transfer(params.swapFeeTo, fee);
examples/src/PaprController.sol::203 => underlying.transfer(proceedsTo, amountOut - fee);
examples/src/PaprController.sol::226 => underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);
examples/src/PaprController.sol::514 => underlying.transfer(params.swapFeeTo, fee);
examples/src/PaprController.sol::515 => underlying.transfer(proceedsTo, amountOut - fee);
examples/src/PaprController.sol::546 => papr.transfer(auction.nftOwner, totalOwed - debtCached);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Unspecific Compiler Version Pragma

#### Impact
Issue Information: [L003](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l003---unspecific-compiler-version-pragma)

#### Findings:
```
examples/src/NFTEDA/NFTEDA.sol::2 => pragma solidity >=0.8.0;
examples/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol::2 => pragma solidity >=0.8.0;
examples/src/NFTEDA/interfaces/INFTEDA.sol::2 => pragma solidity >=0.8.0;
examples/src/NFTEDA/libraries/EDAPrice.sol::2 => pragma solidity >=0.8.0;
examples/src/PaprController.sol::2 => pragma solidity ^0.8.0;
examples/src/PaprToken.sol::2 => pragma solidity ^0.8.0;
examples/src/ReservoirOracleUnderwriter.sol::2 => pragma solidity ^0.8.0;
examples/src/UniswapOracleFundingRateController.sol::2 => pragma solidity ^0.8.0;
examples/src/interfaces/IFundingRateController.sol::2 => pragma solidity >=0.8.0;
examples/src/interfaces/IPaprController.sol::2 => pragma solidity >=0.8.0;
examples/src/interfaces/IUniswapOracleFundingRateController.sol::2 => pragma solidity >=0.8.0;
examples/src/libraries/OracleLibrary.sol::2 => pragma solidity >=0.8.0;
examples/src/libraries/PoolAddress.sol::4 => pragma solidity >=0.8.0;
examples/src/libraries/UniswapHelpers.sol::2 => pragma solidity >=0.8.0;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)