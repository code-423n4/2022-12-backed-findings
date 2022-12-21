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
