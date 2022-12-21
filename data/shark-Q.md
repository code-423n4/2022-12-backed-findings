## 1. Use delete to clear variables instead of zero assignment

A better way to indicate that you are clearing a variable is to use the [`delete`](https://docs.soliditylang.org/en/v0.8.17/types.html#delete) keyword.

For instance:

File: `PaprController.sol` [Line 526](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L526)

```solidity
            _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime = 0;
```

The above instance could be refactored as such:

```solidity
            delete _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime;
```

## 2. Zero address/value checks in the `constructor()`

Zero address/value checks should be present in the `constructor()` to mitigate any human errors. Not doing so could lead to redeploying the whole contract.

For example:

File: `PaprToken.sol` [Line 18-22](https://github.com/with-backed/papr/blob/master/src/PaprToken.sol#L18-L22)

```solidity
    constructor(string memory name, string memory symbol)
        ERC20(string.concat("papr ", name), string.concat("papr", symbol), 18)
    {
        controller = msg.sender;
    }
```

As you can see, no zero address/value checks are made. Moreover, [`controller`](https://github.com/with-backed/papr/blob/master/src/PaprToken.sol#L9) is `immutable`, meaning it cannot be reassigned.

## 3. Insufficient NatSpec

It is recommended that all functions and state variables be thoroughly documented using [NatSpec](https://docs.soliditylang.org/en/develop/natspec-format.html).

The following contract is missing NatSpec in its entirety:

File: [`PaprToken.sol`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol)

## 4. Unspecific Pragma Version

Locking the pragma helps ensure that contracts don't get deployed with unintended versions, for example, the latest compiler which could have higher risks of undiscovered bugs. However, this is present in many files.

## 5, Events are missing old value

Consider having events that set/update state variables have parameters for the old and new values.

The following instances only have parameters for the new value:

- File: `IUniswapOracleFundingRateController.sol` [Line 9](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IUniswapOracleFundingRateController.sol#L9)
- File: `IFundingRateController.sol` [Line 9](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IFundingRateController.sol#L9)

## 6. Typo mistakes

It is recommended to use the proper spelling of words. Typos can make it difficult for readers to decipher the intended meaning, which can lead to confusion.

File: `PaprController.sol` [Line 158](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L158)

```
        /// @audit succesful -> successful
158:    /// @return selector indicating succesful receiving of the NFT
```

File: `NFTEDA.sol` ([Line 44](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L44), [Line 46](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L46), [Line 69](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L69))

```
       /// @audit "does no validation" -> "does not validate"
44:    /// @dev does no validation the auction, aside that it does not exist.
...
       /// @audit defintion -> definition
46:    /// @param auction The defintion of the auction
...
       /// @audit exceed -> exceeds
69:    /// @notice purchases the NFT being sold in `auction`, reverts if current auction price exceed maxPrice

```
