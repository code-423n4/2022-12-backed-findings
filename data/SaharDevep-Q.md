# c4udit Report


### Unsafe ERC20 Operation(s)

#### Impact
Issue Information: [L001](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l001---unsafe-erc20-operations)

#### Findings:
```
src\PaprController.sol::101 => collateralArr[i].addr.transferFrom(msg.sender, address(this), collateralArr[i].id);
src\PaprController.sol::202 => underlying.transfer(params.swapFeeTo, fee);
src\PaprController.sol::203 => underlying.transfer(proceedsTo, amountOut - fee);
src\PaprController.sol::226 => underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);
src\PaprController.sol::514 => underlying.transfer(params.swapFeeTo, fee);
src\PaprController.sol::515 => underlying.transfer(proceedsTo, amountOut - fee);
src\PaprController.sol::546 => papr.transfer(auction.nftOwner, totalOwed - debtCached);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Unspecific Compiler Version Pragma

#### Impact
Issue Information: [L003](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l003---unspecific-compiler-version-pragma)

#### Findings:
```
src\EDAPrice.sol::2 => pragma solidity >=0.8.0;
src\NFTEDAStarterIncentive.sol::2 => pragma solidity >=0.8.0;
src\OracleLibrary.sol::2 => pragma solidity >=0.8.0;
src\PaprController.sol::2 => pragma solidity ^0.8.17;
src\PaprToken.sol::2 => pragma solidity ^0.8.17;
src\PoolAddress.sol::4 => pragma solidity >=0.8.0;
src\ReservoirOracleUnderwriter.sol::2 => pragma solidity ^0.8.17;
src\UniswapHelpers.sol::2 => pragma solidity >=0.8.0;
src\UniswapOracleFundingRateController.sol::2 => pragma solidity ^0.8.17;
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Avoid using low call function ecrecover

#### Impact
[L004]
* In some cases ecrecover can return a random address instead of 0 for an invalid signature. This is prevented above by the owner address inside the typed data.
* Signature are malleable, meaning you might be able to create a second also valid signature for the same data. In our case we are not using the signature data itself (which one may do as an id for example).
* An attacker can construct a hash and signature that look valid if the hash is not computed within the contract itself.

#### Findings:
```
src\ReservoirOracleUnderwriter.sol::68 => address signerAddress = ecrecover(
```

#### Better approach
Use the Openzeppelin contracts. Their ECDSA implementation solves all three problems and they have an EIP-712 implementation (still a draft but usable IMO).


### MISSING CONTRACT-EXISTENCE CHECKS BEFORE LOW-LEVEL CALLS

#### Impact
[L005]
Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that  <address>.code.length > 0


#### Findings:
```
src\ReservoirOracleUnderwriter.sol::68 => address signerAddress = ecrecover(
```

### Use _safeMint instead of _mint

#### Impact
[L006]

#### Findings:
```
src\PaprToken.sol::25 => _mint(to, amount);
```

### Use safeTransfer instead of transfer

#### Impact
[L007]

#### Findings:
```
src\PaprController.sol::202 => underlying.transfer(params.swapFeeTo, fee);
src\PaprController.sol::203 => underlying.transfer(proceedsTo, amountOut - fee);
src\PaprController.sol::514 => underlying.transfer(params.swapFeeTo, fee);
src\PaprController.sol::515 => underlying.transfer(proceedsTo, amountOut - fee);
src\PaprController.sol::546 => papr.transfer(auction.nftOwner, totalOwed - debtCached);

```


### Event is missing indexed fields

#### Impact
[N001]

#### Findings:
```
src\NFTEDAStarterIncentive.sol::21 => event SetAuctionCreatorDiscount(uint256 discount);
```

