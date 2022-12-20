##

## [GAS-1]  instead of using operator && on single require or if checks . using double REQUIRE OR IF checks can save more gas.

BEFORE MIGRATION :

 if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();

As per remix report the execution cost is 21250

Recommended Mitigation Steps:

 if (pool != address(0)) revert PoolTokensDoNotMatch();
 if ( !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();

The execution cost is : 21208

So clearly its possible to save 42 gas 

[File: papr/src/UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol)

     112:   if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();

[File : papr/src/PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

     290:  if (isLastCollateral && remaining != 0) {

##

## [GAS-2]  Its possible to use --X instead of X-=1. --X consumes less gas than X-=1. If we need to increase or decrease value by 1 its better --X/++X . As per remix ide gas reports its possible to save 124 gas for executions.

[File : papr/src/PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

   326:    info.count -= 1;

   419:   _vaultInfo[account][collateral.addr].count += 1;











GAS-1	Use assembly to check for address(0)	2
GAS-2	Using bools for storage incurs overhead	3
GAS-3	Cache array length outside of loop	3
GAS-4	State variables should be cached in stack variables rather than re-reading them from storage	2
GAS-5	Use calldata instead of memory for function arguments that do not get mutated	7
GAS-6	Use Custom Errors	2
GAS-7	Don't initialize variables with default value	3
GAS-8	Using private rather than public for constants, saves gas	1
GAS-9	Use != 0 instead of > 0 for unsigned integer comparison	16
GAS-10	internal functions not called by the contract should be removed	12