## [QA-01] Use stable pragma statement

Using a floating pragma `^0.8.17` statement is discouraged as code can compile to different bytecodes with different compiler versions. Use a stable pragma statement to get a deterministic bytecode.

## [QA-02] Different pragma directives are used

#### Recommendation

Use one Solidity version on each contract.

## [QA-03] USE \_SAFEMINT INSTEAD OF \_MINT

[PaprToken.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L25)

According to openzepplin’s [ERC721](https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-_mint-address-uint256-)

## [QA-04] Avoid using low call function ecrecover

[ReservoirOracleUnderwriter.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L68)

Use OZ library [ECDSA](https://docs.openzeppelin.com/contracts/4.x/api/utils#ECDSA) that its battle tested to avoid classic errors.

## [QA-05] Constants should be defined rather than using magic numbers

PaprController.sol

```
92:        address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);
```
