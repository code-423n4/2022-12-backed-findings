## 1.  Duplicate computation of `price` variable

In the `_auctionCurrentPrice` function, the price variable is computed twice in case of `msg.sender == auctionState[id].starter`

# Proof Of Concept:
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L60
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L63

# Recommendation: 
Use if-else statement
```
        if (msg.sender == auctionState[id].starter) {
            price = FixedPointMathLib.mulWadUp(price, _pricePercentAfterDiscount);
        }
        else{
            super._auctionCurrentPrice(id, startTime, auction);
        }
```

## 2.  State variables declared `immutable` instead of `constant`

State variables are declared immutable incase, we need to initialize these variables once but the below shown variables have already been initialized and are not being updated later in the code as well. 
# Proof Of Concept:

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L41
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L44
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L47
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L50
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L54


# Recommendation: 

Declare these variables as `constant`