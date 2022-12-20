# QA report

## Low Risk
| L-R    |Issue|Instances|
|:------:|:----|:-------:|
| [L&#x2011;01] | Missing zero address check in the constructor | 1 |
| [L&#x2011;02] | Unsafe ERC20 operations | 3 |
| [L&#x2011;03] | decimals() not part of ERC20 standard  | 1 |
| [L&#x2011;04] | Signature malleability for `ecrecover`  | 1 |

Total: 6 instances over 4 issues

## Non-critical

| N-C   |Issue|Instances|
|:------:|:----|:-------:|
| [N&#x2011;01] | 10 ** 18 can be 1e18 for readability | 2 |
| [N&#x2011;02] | Constants should be defined rather than using magic numbers | 2 |
| [N&#x2011;03] | Immutable must be changed to constant | 5 |
| [N&#x2011;04] | Typos | 1 |
| [N&#x2011;05] | Change name of a parameter | 1 |
| [N&#x2011;06] | Do not use floating pragma, use a concrete version instead | 5 |
| [N&#x2011;07] | Lack of event emitting | 1 |

Total: 17 instances over 7 issues

## Low Risk

### \[L-01\] Missing zero address check in the constructor

File: `PaprController.sol`

The variable `oracleSigner` of type address in the constructor is initialized in `ReservoirOracleUnderwriter.sol`  where zero address check is missing.

```solidity
constructor(
        string memory name,
        string memory symbol,
        uint256 _maxLTV,
        uint256 indexMarkRatioMax,
        uint256 indexMarkRatioMin,
        ERC20 underlying,
        address oracleSigner
```

Line of code:

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L74

Consider adding a zero address checks.

### \[L-02\] Unsafe ERC20 operations

Some tokens do not comply with the standard. The transfer() function may have no return value, such as USDT. In such cases, the payout functions will revert and users fund may be locked.

Although the SafeTransferLib is imported in `PaprController.sol` , the `transfer()` function should be converted to `safeTransfer()`


```solidity
underlying.transfer(proceedsTo, amountOut - fee);
```

```solidity
underlying.transfer(params.swapFeeTo, amountIn * params.swapFeeBips / BIPS_ONE);
```

```solidity
collateralArr[i].addr.transferFrom(msg.sender, address(this), collateralArr[i].id);
```

Lines of code:

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L203

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L226

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L101


###  \[L-03\] decimals() not part of ERC20 standard

decimals() is not part of the official ERC20 standard and might fall for tokens that do not implement it. While in practice it is very unlikely, as usually most of the tokens implement it, this should still be considered as a potential issue.

```solidity
uint256 underlyingONE = 10 ** underlying.decimals();
```

Line of code:

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L82


###  \[L-04\]  Signature malleability for ecrecover `ecrecover` 

File: `ReservoirOracleUnderwriter.sol`

The implementation of a cryptographic signature system in Ethereum contracts often assumes that the signature is unique, but signatures can be altered without the possession of the private key and still be valid. The EVM specification defines several so-called ‘precompiled’ contracts one of them being ecrecover which executes the elliptic curve public key recovery. A malicious user can slightly modify the three values v, r and s to create other valid signatures. A system that performs signature verification on contract level might be susceptible to attacks if the signature is part of the signed message hash. Valid signatures could be created by a malicious user to replay previously signed messages.

```solidity
address signerAddress = ecrecover(
```

Line of code: 

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L68


Use OpenZeppelin ECDSA library.


## Non-critical

###  \[NC-01\] 10 ** 18 can be 1e18 for readability

File: `PaprController.sol`

```solidity
initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(underlyingONE, 10 ** 18);
```

```solidity
initSqrtRatio = UniswapHelpers.oneToOneSqrtRatio(10 ** 18, underlyingONE);
```

Lines of code:

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L87

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L89


### \[NC-02\] Constants should be defined rather than using magic numbers

File: `PaprController.sol`

```solidity
 if (newDebt >= 1 << 200) revert IPaprController.DebtAmountExceedsUint200();
```

```solidity
address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);
```

Lines of code:

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L92

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L473


###  \[NC-03\] Immutable must be changed to constant

File: `PaprController.sol`

Immutable must be changed to constant for some variables because they are not initialized in the constructor but before that


```solidity
 uint256 public immutable override liquidationAuctionMinSpacing = 2 days;
  
 uint256 public immutable override perPeriodAuctionDecayWAD = 0.7e18;

 uint256 public immutable override auctionDecayPeriod = 1 days;

 uint256 public immutable override auctionStartPriceMultiplier = 3;

 uint256 public immutable override liquidationPenaltyBips = 1000;
```

Lines of code:

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L41

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L44

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L47

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L50

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L54

###  \[NC-04\] Typos 

File: `PaprController.sol`

```solidity
/// @return selector indicating `succesful` receiving of the NFT
```

Change it to `successful`

Line of code:

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L158


###  \[NC-05\] Change the name of  `totalCollateraValue`  parameter

File: `PaprController.sol` and `IPaprController.sol`

The current name of the parameter is `totalCollateraValue` , it will be more professional to change it with the correct spelling - totalCollateralValue

```solidity
 function maxDebt(uint256 totalCollateraValue) external view override returns (uint256) {
```

Line of code:

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L393

###  \[NC-06\] Do not use floating pragma, use a concrete version instead

Currently the smart contracts are using a floating pragma, change it to concrete version

###  \[NC-07\] Lack of event emitting 

Contracts do not emit relevant event on setter function.
Consider emitting events after sensitive changes take place, to facilitate tracking and notify off-chain clients following the contract’s activity.

File: `PaprController.sol`



```solidity
function setLiquidationsLocked(bool locked) external override onlyOwner {
        liquidationsLocked = locked;
    }
```

Lines of code:

- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L360