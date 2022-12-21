# QA Report

## Summary

|               | Issue         | Risk     | Instances     |
| :-------------: |:-------------|:-------------:|:-------------:|
| 1     | Owner is unable to change `auctionCreatorDiscountPercentWad` value after deployment | Low | 1 |
| 2     | Add a check to ensure that `targetMarkRatioMax > targetMarkRatioMin`  | Low | 1 |
| 3     | `setLiquidationsLocked` function should emit an event  | Low | 1 |
| 4     | Non-library/interface files should use fixed compiler versions, not floating ones | NC | 5  |
| 5     | Use scientific notation | NC | 4 |


## Findings

### 1- Owner is unable to change `auctionCreatorDiscountPercentWad` value after deployment :

The value of the state variables `auctionCreatorDiscountPercentWad` and `_pricePercentAfterDiscount` is used to determine the discount in the auction price given to the auction starter as incentive, but this value can not be changed after the deployment of the PaprController contract as there is not external/public function to do so. 

Even if the creator discount value was not intented to be changed in the protocol concepts, you should consider adding a setter function which will allow only the owner to update it (just in case it was needed in the future).

#### Risk : Low

#### Proof of Concept

The values of both `auctionCreatorDiscountPercentWad` and `_pricePercentAfterDiscount` are only set in the constructor of `NFTEDAStarterIncentive` contract :

```
constructor(uint256 _auctionCreatorDiscountPercentWad) {
    _setCreatorDiscount(_auctionCreatorDiscountPercentWad);
}
```

And `_setCreatorDiscount` is an internal function and there is no external/public function in the PaprController contract that calls it.

#### Mitigation

Consider adding an external function callable only by the onwer to allow the change of the creator discount value in the future.

### 2- Add a check to ensure that `targetMarkRatioMax > targetMarkRatioMin` :

The state variables `targetMarkRatioMax` and `targetMarkRatioMin` are immutable and their values are set in the consturctor of `UniswapOracleFundingRateController`, those values are used to limit the value of (target/mark) and are essentiel for the protocol working, so when setting them the constructor should check that `targetMarkRatioMax` is always greater than  `targetMarkRatioMin` as there values can't be changed later on.

#### Risk : Low

#### Proof of Concept

The `UniswapOracleFundingRateController` constructor doesn't check the input values of both `_targetMarkRatioMax` and  `_targetMarkRatioMin` variables :

```
constructor(ERC20 _underlying, ERC20 _papr, uint256 _targetMarkRatioMax, uint256 _targetMarkRatioMin) {
    underlying = _underlying;
    papr = _papr;

    targetMarkRatioMax = _targetMarkRatioMax;
    targetMarkRatioMin = _targetMarkRatioMin;

    _setFundingPeriod(4 weeks);
}
```

#### Mitigation

You should add a check statement to make sure that you always have `targetMarkRatioMax > targetMarkRatioMin`, the constructor should be modified as follow :

```
constructor(ERC20 _underlying, ERC20 _papr, uint256 _targetMarkRatioMax, uint256 _targetMarkRatioMin) {
    underlying = _underlying;
    papr = _papr;

    require(_targetMarkRatioMax > _targetMarkRatioMin);
    targetMarkRatioMax = _targetMarkRatioMax;
    targetMarkRatioMin = _targetMarkRatioMin;

    _setFundingPeriod(4 weeks);
}
```

### 3- `setLiquidationsLocked` function should emit an event :

The function `setLiquidationsLocked` is used to change the boolean variable `liquidationsLocked` which determine if auctions can be started or not, as this parameter is really critical for the protocol working this function should emit an event to allow all users to know when auctions are possible. 

#### Risk : Low

#### Proof of Concept

The function `setLiquidationsLocked` doesn't emit an event :

File: src/PaprController.sol [Line 360-362](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L360-L362)
```
function setLiquidationsLocked(bool locked) external override onlyOwner {
    liquidationsLocked = locked;
}
```

#### Mitigation

Emit an event in the `setLiquidationsLocked` function.

### 4- Non-library/interface files should use fixed compiler versions, not floating ones :

Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.

check this source : https://swcregistry.io/docs/SWC-103

#### Risk : NON CRITICAL

#### Proof of Concept

There followings contract all use a floating solidity version `pragma solidity ^0.8.17` :

File: [src/PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

File: [src/PaprToken.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol)

File: [src/ReservoirOracleUnderwriter.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol)

File: [src/UniswapOracleFundingRateController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol)

File: [src/NFTEDA/NFTEDA.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol)

#### Mitigation
Lock the pragma version to the same version as used in the other contracts and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.

Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. Otherwise, the developer would need to manually update the pragma in order to compile it locally.

### 5- Use scientific notation :

When using multiples of 10 you shouldn't use decimal literals or exponentiation (e.g. 1000000, 10**18) but use scientific notation for better readability.

#### Risk : NON CRITICAL

#### Proof of Concept

Instances include:

File: src/PaprController.sol [Line 54](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L54)

```
uint256 public immutable override liquidationPenaltyBips = 1000;
```

File: src/PaprController.sol [Line 87](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L87)

```
initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);
```

File: src/PaprController.sol [Line 89](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L89)

```
initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);
```

File: src/PaprController.sol [Line 92](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L92)

```
address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);
```

#### Mitigation

Use scientific notation for the instances aforementioned.