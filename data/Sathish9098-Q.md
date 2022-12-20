##

## [NC-1]  USE A MORE RECENT VERSION OF SOLIDITY

[File: papr/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol)

     2:  pragma solidity >=0.8.0;

##

## [NC-2] Shorter inheritance list

[File: papr/src/UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol)

    contract PaprController is
    IPaprController,
    UniswapOracleFundingRateController,
    ERC721TokenReceiver,
    Multicallable,
    Ownable2Step,
    ReservoirOracleUnderwriter,
    NFTEDAStarterIncentive

##
 
## [NC-3]  CONSTANTS SHOULD BE DEFINED RATHER THAN USING MAGIC NUMBERS

Even assembly can benefit from using readable constants instead of hex/numeric literals

[File : papr/src/PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

    87:  initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);

    89:   initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);

##

## [NC-4]  NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES

instances(8): 

[IUniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IUniswapOracleFundingRateController.sol)

    2:  pragma solidity >=0.8.0;

##

## [NC-5]  Use same versions for all contracts . Now contracts using different solidity versions. Some contracts using latest versions and some contracts are using very old versions.

##

## [L-1] collateralArr array length must be checked. Its possible to call addCollateral() function with empty array. To avoid this we need to check the collateralArr length before entering into the loop executions.

[File : papr/src/PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

[PaprController.sol#L98-L106)](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L98-L106)

Recommended mitigation :

require(collateralArr.length >0);











NC-1	Missing checks for address(0) when assigning values to address state variables	3
NC-2	require() / revert() statements should have descriptive reason strings	11
NC-3	Event is missing indexed fields	7
NC-4	Functions not used internally could be marked external

L-1	Unsafe ERC20 operation(s)	7
L-2	Unspecific compiler version pragma	10