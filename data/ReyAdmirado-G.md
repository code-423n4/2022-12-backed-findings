
| | issue |
| ----------- | ----------- |
| 1 | [state variables should be cached in stack variables rather than re-reading them from storage](#) |
| 2 | [Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement](#) |
| 3 | [`<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables](#) |
| 4 | [internal functions only called once can be inlined to save gas](#) |
| 5 | [abi.encode() is less efficient than abi.encodepacked()](#) |
| 6 | [usage of uint/int smaller than 32 bytes (256 bits) incurs overhead](#) |
| 7 | [public functions not called by the contract should be declared external instead](#) |
| 8 | [before some functions we should check some variables for possible gas save](#) |

## 1. state variables should be cached in stack variables rather than re-reading them from storage (ones that was not caught in known issues)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

cache `pool` before the if statement because its used twice in its condition part and `pool` will be read twice
- [UniswapOracleFundingRateController.sol#L112](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L112)


## 2. Add `unchecked {}` for subtractions where the operands cannot underflow because of a previous `require()` or `if` statement

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }
if(a <= b); x = b - a => if(a <= b); unchecked { x = b - a }
this will stop the check for overflow and underflow so it will save gas

- [PaprController.sol#L546](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L546)
- [PaprController.sol#L550](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L550)


## 3. `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables
Using the addition operator instead of plus-equals saves gas

- [PaprController.sol#L326](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326)
- [PaprController.sol#L419](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L419)


## 4. internal functions only called once can be inlined to save gas

Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

`_startAuction` is only used once in paprController.sol
- [NFTEDA.sol#L48](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L48)

`_setCreatorDiscount`
- [NFTEDAStarterIncentive.sol#L72](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L72)

`_clearAuctionState` only used once in NFTEDA.sol
- [NFTEDAStarterIncentive.sol#L48](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L48)

`_setAuctionStartTime`only used once in NFTEDA.sol
- [NFTEDAStarterIncentive.sol#L43](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L43)

`auctionStartTime`only used once in NFTEDA.sol
- [NFTEDAStarterIncentive.sol#L38](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L38)

`_increaseDebtAndSell`
- [PaprController.sol#L493](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L493)

`_purchaseNFTAndUpdateVaultIfNeeded`
- [PaprController.sol#L519](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L519)

`_handleExcess`
- [PaprController.sol#L532](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L532)


## 5. abi.encode() is less efficient than abi.encodepacked()

- [PaprController.sol#L197](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L197)
- [PaprController.sol#L222](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L222)
- [PaprController.sol#L509](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L509)


## 6. usage of uint/int smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html Use a larger size then downcast where needed

- [PaprController.sol#L83](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L83)
- [PaprController.sol#L436](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L436)


## 7. public functions not called by the contract should be declared external instead

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

`auctionCurrentPrice`
- [NFTEDA.sol#L24](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L24)

`underwritePriceForCollateral`
- [ReservoirOracleUnderwriter.sol#L64](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L64)

`updateTarget`
- [UniswapOracleFundingRateController.sol#L45](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L45)

`newTarget`
- [UniswapOracleFundingRateController.sol#L63](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L63)

`mark`
- [UniswapOracleFundingRateController.sol#L72](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L72)


## 8. before some functions we should check some variables for possible gas save

before transfer we should check for amount being 0 so the function doesnt run when its not gonna do anything

`price`
- [NFTEDA.sol#L93](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L93)

`amount`
- [PaprController.sol#L383](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L383)
