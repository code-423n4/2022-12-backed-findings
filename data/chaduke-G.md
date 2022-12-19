G1. https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L81
Much code is depending on the following line
```
        token0IsUnderlying = address(underlying) < address(papr);
```
At the same time, the truth value is fixed once ``underlying`` and ``papr`` contract have been deployed. 
To save gas, we suggest that the ``papr`` should be deployed first, and then the ``underlying`` contract can
be defined as a constant. As a result,  the value of ``token0IsUnderlying`` can be predetermined. In this way, 
many if the if-else statement based on the truth value of `` token0IsUnderlying`` can be simplified to save gas
as well as improving the readability of the code. 

G2. https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L121
Calculating ``oraclePrice`` in each iteration repeatedly is a waste of gas. We only need to calculate ``oraclePrice`` to save gas.
```
function removeCollateral(
        address sendTo,
        IPaprController.Collateral[] calldata collateralArr,
        ReservoirOracleUnderwriter.OracleInfo calldata oracleInfo
    ) external override {
        uint256 cachedTarget = updateTarget();
        uint256 oraclePrice;
        ERC721 collateralAddr;

        collateralAddr = collateralArr[0].addr;
        oraclePrice =    underwritePriceForCollateral(collateralAddr, ReservoirOracleUnderwriter.PriceKind.LOWER, oracleInfo);
        _removeCollateral(sendTo, collateralArr[0], oraclePrice, cachedTarget);
   


        for (uint256 i = 1; i < collateralArr.length;) {
                if (collateralAddr != collateralArr[i].addr) {
                    revert CollateralAddressesDoNotMatch();
                }
              _removeCollateral(sendTo, collateralArr[i], oraclePrice, cachedTarget);

            unchecked {
                ++i;
            }
        }
    }
```