# 1. inadequate NatSpec

Solidity contracts can use the Ethereum Natural Language Specification Format (NatSpec) to provide detailed documentation for functions, return variables, and other elements of the contract. This is done using a special type of comment within the contract code.

Here is an example of a NatSpec comment in a Solidity contract:

```solidity
/// @notice This is a NatSpec comment that provides
/// a brief description of the `mint` function.
function mint(address to, uint256 amount) external onlyController {
  ...
}
```

NatSpec comments start with `///` and follow a specific format that includes tags such as `@notice` and `@param` to indicate the type of information being provided. You can find more information about NatSpec and its usage in Solidity contracts at the following link:

- [NatSpec Format](https://docs.soliditylang.org/en/v0.8.16/natspec-format.html)

This is present in 2 files:

- [PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)
- [PaprToken.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol)

# 2. Typos

1. [PaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol)

   - [PaprController.sol#L158](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L158), [PaprController.sol#L393](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L393), [PaprController.sol#L556](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L556)

     ```solidity
     line 158:    /// @return selector indicating succesful receiving of the NFT

     line 393:    function maxDebt(uint256 totalCollateraValue) external view override returns (uint256) {

     line 556:    function _maxDebt(uint256 totalCollateraValue, uint256 cachedTarget) internal view returns (uint256) {
     ```

   - Consider making the following changes:
     - `succesful` To `successful`
     - `totalCollateraValue` To `totalCollateralValue`

2. [NFTEDA.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol)

   - [NFTEDA.sol#L46](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L46)

     ```solidity
     line 46:    /// @param auction The defintion of the auction
     ```

     - Consider changing `defintion` To `definition`

3. [INFTEDA.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/interfaces/INFTEDA.sol)

   - [INFTEDA.sol#L58](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/interfaces/INFTEDA.sol#L58)

     ```solidity
     line 58:    /// @dev Derived from the auction. Identitical auctions cannot exist simultaneously
     ```

     - Consider changing `Identitical` To `Identical`

4. [IPaprController.sol](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol)

   - [IPaprController.sol#L266](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L266)

     ```solidity
     line 266:    function maxDebt(uint256 totalCollateraValue) external view returns (uint256);
     ```

     - Consider changing `totalCollateraValue` To `totalCollateralValue`


# 3. Use the `delete` operator to clear variables, instead of assigning a value of zero.

To clear variables, use the `delete` operator rather than assigning to zero. This conveys the intention more clearly and is more idiomatic.

There is one instance where this can be implemented in the code:

- [PaprController.sol#L526](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L526)
