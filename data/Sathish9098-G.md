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

Instances (2) :

[File: papr/src/UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol)

     112:   if (pool != address(0) && !UniswapHelpers.poolsHaveSameTokens(pool, _pool)) revert PoolTokensDoNotMatch();

[File : papr/src/PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

     290:  if (isLastCollateral && remaining != 0) {

##

## [GAS-2]  Its possible to use --X instead of X-=1. --X consumes less gas than X-=1. If we need to increase or decrease value by 1 its better --X/++X . As per remix ide gas reports its possible to save 124 gas for executions.

Instances(2) :

[File : papr/src/PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

   326:    info.count -= 1;

   419:   _vaultInfo[account][collateral.addr].count += 1;

## [GAS-3] Use uint256 instead of uint16 . uint256 consumes less gas than uint16 

A smart contract's gas consumption can be higher if developers use items that are less than 32 bytes in size because the Ethereum Virtual Machine can only handle 32 bytes at a time. In order to increase the element's size to the necessary size, the EVM has to perform additional operations. 

[File : papr/src/PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

      436:  uint16 newCount;











