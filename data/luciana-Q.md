The `from` variable can be set `address(0)` in `onERC721Received(...) function.

[`PaprController.sol#L159`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L159)

```solidity 
function onERC721Received(address from, address, uint256 _id, bytes calldata data)
```

*Recommendation:* Consider adding `address(0)` check for `_oracleSigner` and  `_quoteCurrency` variables.

---
## [Low] Owner could send `papr` tokens to `address(0)`

The `sendPaprFromAuctionFees` function does not validate the `to` address. Despite, this function can be called only by the owner, `papr` token could be transfered to `address(0)` as mistake. 

[`PaprController.sol#L382`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L382)

```solidity
    function sendPaprFromAuctionFees(address to, uint256 amount) external override onlyOwner {
        papr.safeTransferFrom(address(this), to, amount);
    }
```
*Recommendation: * The function should check if the `to` local variable is a valid address.

---
## [Info] Missing `address(0)` checks

The `onERC721Received(...)` function in `PaprController` contract should validate: 
- `from` 
- `request.proceedsTo`
- `request.swapParams.swapFeeTo`




```solidity
function onERC721Received(address from, address, uint256 _id, bytes calldata data) 
```



 `_oracleSigner` and  `_quoteCurrency` variables in  could be set as `address(0)` as mistake that could NOT be fixed later.

[`ReservoirOracleUnderwriter.sol#L51-L54`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L51-L54)

The constructor is presented below:

```solidity
    constructor(address _oracleSigner, address _quoteCurrency) {
        oracleSigner = _oracleSigner;
        quoteCurrency = _quoteCurrency;
    }
```

---