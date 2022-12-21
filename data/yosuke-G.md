## [YO GAS-1] Using private rather than public for constants and immutable, saves gas

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L9
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L35-L44
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L17-L19
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L25-L27
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L30
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L35-L54

### Recommended Mitigation Steps
Use private instead of public for constants and immutable.

## [YO GAS-2] Use != 0 instead of > 0 for unsigned integer comparisons to save gas.

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L170-L174
```solidity=
        if (request.swapParams.minOut > 0) {} 
        else if (request.debt > 0) {}
```

### Recommended Mitigation Steps
Use != 0 instead of > 0 for unsigned integer comparisons to save gas.
```solidity=
        if (request.swapParams.minOut != 0) {
            _increaseDebtAndSell(from, request.proceedsTo, ERC721(msg.sender), request.swapParams, request.oracleInfo);
        } else if (request.debt != 0) {
            _increaseDebt(from, collateral.addr, request.proceedsTo, request.debt, request.oracleInfo);
        }
```

## [YO GAS-3] USING BOOLS FOR STORAGE INCURS OVERHEAD

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L188-L204
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L213


### Recommended Mitigation Steps
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas) for the extra SLOAD, and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past.
```solidity=
    function increaseDebtAndSell(
        address proceedsTo,
        ERC721 collateralAsset,
        IPaprController.SwapParams calldata params,
        ReservoirOracleUnderwriter.OracleInfo calldata oracleInfo
    ) external override returns (uint256 amountOut) {
        uint256 hasFee;
        if(params.swapFeeBips != 0) hasFee = 2;
        else hasFee = 1;

        (amountOut,) = UniswapHelpers.swap(
            pool,
            hasFee == 2 ? address(this) : proceedsTo,
            !token0IsUnderlying,
            params.amount,
            params.minOut,
            params.sqrtPriceLimitX96,
            abi.encode(msg.sender, collateralAsset, oracleInfo)
        );

        if (hasFee == 2) {
            uint256 fee = amountOut * params.swapFeeBips / BIPS_ONE;
            underlying.transfer(params.swapFeeTo, fee);
            underlying.transfer(proceedsTo, amountOut - fee);
        }
    }
```
```solidity=
    /// @inheritdoc IPaprController
    function buyAndReduceDebt(address account, ERC721 collateralAsset, IPaprController.SwapParams calldata params)
        external
        override
        returns (uint256)
    {
        uint256 hasFee;
        if(params.swapFeeBips != 0) hasFee = 2;
        else hasFee = 1;

        (uint256 amountOut, uint256 amountIn) = UniswapHelpers.swap(
            pool,
            account,
            token0IsUnderlying,
            params.amount,
            params.minOut,
            params.sqrtPriceLimitX96,
            abi.encode(msg.sender)
        );

        if (hasFee == 2) {
            underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);
        }

        _reduceDebt({account: account, asset: collateralAsset, burnFrom: msg.sender, amount: amountOut});

        return amountOut;
    }
```

