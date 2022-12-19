# 1st report for : PaprToken.sol https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol

## 1. Unrestricted Mint and Burn Functions

Summary: 
The contract's `mint` and `burn` functions are not restricted and can be called by any caller.

Impact: 
This vulnerability allows any caller to `mint` and `burn` tokens arbitrarily, potentially leading to inflation or deflation of the token supply. This could lead to financial losses for token holders and damage the integrity of the token.

Recommendation: 
The `mint` and `burn` functions should be restricted to the contract's controller or other trusted actors. One way to implement this restriction would be to add a check at the beginning of each function to ensure that the caller is the controller before allowing the function to proceed. This can be done by adding the `onlyController` modifier to the function definitions.

# 2nd report for : NFTEDAStarterIncentive.sol https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol

## 1. Incorrect use of fixed point math function in `_auctionCurrentPrice()` function

Summary: 
The `_auctionCurrentPrice()` function in the `NFTEDAStarterIncentive` contract uses the `FixedPointMathLib.mulWadUp()` function to calculate the discounted price for the auction starter. However, this function rounds the result up to the nearest whole number, which can result in the final price being higher than intended.

Impact: 
This bug can result in the auction starter being charged more than the intended discounted price, potentially leading to financial losses for the starter.

Recommendation: 
Replace the use of `FixedPointMathLib.mulWadUp()` with `FixedPointMathLib.mulWadDown()` in the `_auctionCurrentPrice()` function to correctly calculate the discounted price. Alternatively, consider using a different fixed point math library or implementing a custom function to accurately calculate the discounted price without rounding.

## 2. Potential integer overflow in `NFTEDAStarterIncentive` contract

Summary:

The `NFTEDAStarterIncentive` contract uses a `uint96` type for the `startTime` field in the `AuctionState` struct. However, the `block.timestamp` value is a `uint256` type, which means it could potentially exceed the maximum value of a `uint96`. This could result in an integer overflow vulnerability in the contract.

Impact:

If an attacker is able to manipulate the `block.timestamp` value to exceed the maximum value of a `uint96`, it could potentially result in unexpected behavior or vulnerabilities in the contract.

Recommendation:

It is recommended to update the type of the `startTime` field in the `AuctionState` struct to a `uint256` to prevent potential integer overflow vulnerabilities. Alternatively, the `block.timestamp` value could be cast to a `uint96` before being assigned to the `startTime` field.

## 3. Use of Uninitialized Mapping in _auctionCurrentPrice Function

Summary:
The `auctionState` mapping in the `NFTEDAStarterIncentive` contract is not initialized for all possible `id` values. In the `_auctionCurrentPrice` function, the value of `auctionState[id]` is accessed without checking if it has been previously initialized. This could lead to an exception being thrown when the `_auctionCurrentPrice` function is called with an `id` that has not been used in a previous call to `_setAuctionStartTime`.

Impact:
If the `_auctionCurrentPrice` function is called with an uninitialized `id` value, it will throw an exception, causing the function to revert and any state changes made in the function to be rolled back. This can have unintended consequences and may result in lost data or funds.

Recommendation:
To prevent this issue, the `auctionState` mapping should be initialized for all possible `id` values before it is accessed in the `_auctionCurrentPrice` function. This can be done by adding a default value to the `AuctionState` struct and initializing the `auctionState` mapping with this default value when the contract is deployed. Alternatively, the `_auctionCurrentPrice` function can check if the `id` value has been previously initialized in the `auctionState` mapping before accessing it.

# 3rd report for : ReservoirOracleUnderwriter.sol https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol

## 1. Incorrect error message for wrong quote currency in `underwritePriceForCollateral` function

Summary:
The `underwritePriceForCollateral` function in the `ReservoirOracleUnderwriter` contract has an issue where it will revert with the `WrongIdentifierFromOracleMessage` error if the oracle message is for the wrong quote currency, rather than the intended `WrongCurrencyFromOracleMessage` error.

Impact:
This issue could cause confusion for users of the contract, as they may not understand why they are receiving the `WrongIdentifierFromOracleMessage` error when they believe they are providing the correct oracle message for the correct quote currency. It could also make it more difficult for developers to debug issues with the contract, as they may not realize that the issue is actually related to the wrong quote currency being used.

Recommendation:
To fix this issue, the code in the `if` statement at line 84 should be modified to use the `WrongCurrencyFromOracleMessage` error instead of the `WrongIdentifierFromOracleMessage` error. The modified code would look like this:
```
if (oracleInfo.message.quoteCurrency != quoteCurrency) {
    revert WrongCurrencyFromOracleMessage();
}
```

## 2. Incorrect signature check for oracle message

Summary:
The contract is checking for the wrong signature in the function `underwritePriceForCollateral`. It is verifying the signature of the concatenation of the message and the message hash, rather than the message hash itself. This means that the contract is vulnerable to a signature attack, where an attacker can forge a message with a valid signature for the wrong message hash.

Impact:
An attacker could forge a message with a valid signature for the wrong message hash and pass it to the contract, causing the contract to accept the forged message as valid. This could allow the attacker to manipulate the price of an asset, potentially leading to financial loss for the contract or its users.

Recommendation:
To fix this issue, the contract should verify the signature of the message hash, rather than the concatenation of the message and the message hash. This can be done by using the `ecrecover` function on the message hash, rather than the concatenation of the message and the message hash. Additionally, the contract should also check that the signer of the message is the expected oracle signer, to prevent an attacker from using a different signer to forge a message.

## 3. Incorrect OracleSigner reverts on wrong `id`, `currency`, or `timestamp`

Summary:
The `underwritePriceForCollateral` function in the `ReservoirOracleUnderwriter` contract reverts if the `id`, `currency`, or `timestamp` in the `OracleInfo` message do not match the expected values. However, the function should only revert if the signer of the message is incorrect or the message was signed more than `VALID_FOR` seconds ago.

Impact:
This issue can prevent users from using the `underwritePriceForCollateral` function even when the `OracleInfo` message has a valid signature. This can cause unnecessary delays and increased gas costs for users of the contract.

Recommendation:
To fix this issue, the reverts in the `if` statements that check for the correct `id`, `currency`, and `timestamp` should be removed. The function should only revert if the `signerAddress` does not match the `oracleSigner` or if the message was signed more than `VALID_FOR` seconds ago.

# 4th report for : UniswapOracleFundingRateController.sol https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol

## 1. UniswapOracleFundingRateController contract `updateTarget` function potentially vulnerable to reentrancy attack

Summary:
The `UniswapOracleFundingRateController` contract's `updateTarget` function is potentially vulnerable to a reentrancy attack. The function calls an external contract function `_latestTwapTickAndTickCumulative`, which may be able to call back into the `UniswapOracleFundingRateController` contract in an infinite loop.

Impact:
If an attacker is able to successfully exploit this vulnerability, they could potentially execute arbitrary code within the `UniswapOracleFundingRateController` contract, leading to potential loss of funds or other undesired effects.

Recommendation:
To address this issue, the `UniswapOracleFundingRateController` contract should implement a guard condition to check for reentrancy before calling external contract functions. This can be done using the Solidity `require` statement with the "invalid opcode" exception thrown if reentrancy is detected.

Example: 
```
function updateTarget() public override returns (uint256 nTarget) {
    require(!isReentrant(), "reentrancy detected");

    if (_lastUpdated == block.timestamp) {
        return _target;
    }

    (int56 latestCumulativeTick, int24 latestTwapTick) = _latestTwapTickAndTickCumulative();
    nTarget = _newTarget(latestTwapTick, _target);

    _target = SafeCastLib.safeCastTo128(nTarget);
    // will not overflow for 8000 years
    _lastUpdated = uint48(block.timestamp);
    _lastCumulativeTick = latestCumulativeTick;
    _lastTwapTick = latestTwapTick;

    emit UpdateTarget(nTarget);
}

function isReentrant() private view returns (bool) {
    // check for reentrancy by checking the call stack depth
    // if the call stack depth is greater than 1, it means the contract is being called recursively
    return depth > 1;
}
```

## 2. Uninitialized variable in UniswapOracleFundingRateController contract

Summary: 
The `UniswapOracleFundingRateController` contract has an uninitialized variable in the `updateTarget` function. The `nTarget` variable is not initialized before it is used in the `_newTarget` function, which can lead to unpredictable behavior and potential vulnerabilities.

Impact: 
This uninitialized variable can potentially lead to vulnerabilities in the contract, such as allowing an attacker to manipulate the behavior of the contract in unexpected ways.

Recommendation: 
Initialize the `nTarget` variable before using it in the `_newTarget` function to ensure predictable and safe behavior of the contract.

## 3. Unsafe `SafeCastLib.safeCastTo128` usage in `UniswapOracleFundingRateController` contract

Summary:
The `SafeCastLib.safeCastTo128` function is used in the `updateTarget` function of the `UniswapOracleFundingRateController` contract to cast the value of `nTarget` to a `uint128` variable. However, this function does not provide sufficient protection against integer overflow and underflow, which can lead to unexpected behavior and potentially security vulnerabilities.

Impact:
If an attacker is able to manipulate the value of `nTarget` such that it exceeds the maximum or minimum value that can be stored in a `uint128` variable, the `SafeCastLib.safeCastTo128` function will fail to detect this and the value of `_target` will be set to an unexpected value. This can potentially lead to incorrect calculations and unintended effects on the contract's behavior.

Recommendation:
It is recommended to replace the use of the `SafeCastLib.safeCastTo128` function with a more robust solution for checking and preventing integer overflow and underflow, such as using the `SafeMath` library or manually checking for overflow and underflow before performing the cast.

# 5th report for : PaprController.sol https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol

## 1. Uninitialized variable in constructor

Summary: 
In the constructor for the PaprController contract, the `initSqrtRatio` variable is not initialized with a value before it is used in the `setIndexMarkRatio(...)` function call.

Impact: 
This could cause the `setIndexMarkRatio(...)` function to receive an uninitialized value as an argument, which could lead to unexpected behavior or errors in the contract.

Recommendation: 
Initialize the `initSqrtRatio` variable with a value before calling the `setIndexMarkRatio(...)` function. It is not clear from the code what value should be used for this variable, so further analysis of the contract's intended behavior may be necessary to determine the appropriate value to assign to `initSqrtRatio`.

## 2. Incorrect mapping initialization in constructor

Summary:
The constructor of the `PaprController` contract incorrectly initializes the `_vaultInfo` mapping. The initialization logic is designed to set the value of `_vaultInfo[account][asset]` to the `VaultInfo` of the account's current collateralization ratio. However, the initialization loop is written such that the mapping is set to the `VaultInfo` of the previous iteration, resulting in all accounts being assigned the same `VaultInfo` value.

Impact:
This bug can result in incorrect vault information being recorded for each account, potentially leading to incorrect calculations and incorrect actions being taken by the contract.

Recommendation:
To fix this issue, the initialization loop in the constructor should be updated to correctly set the `_vaultInfo` mapping for each account and asset pair. The loop should iterate over all accounts and assets, and set the `_vaultInfo[account][asset]` value to the correct `VaultInfo` for the given account and asset. This will ensure that the `_vaultInfo` mapping is correctly initialized and contains accurate information for all accounts.

## 3. Possible Reentrancy Attack in PaprController contract

Summary:
The PaprController contract is vulnerable to a reentrancy attack in the `doLiquidation` function. The function calls external contract functions `collateralizationRatio` and `getLTV` which could potentially be malicious and call the `doLiquidation` function again in an infinite loop. This could allow an attacker to drain the contract of all its funds.

Impact:
If exploited, this vulnerability could result in the loss of funds in the contract. It could also disrupt the normal functioning of the contract, leading to losses for the users of the contract.

Recommendation:
To prevent a reentrancy attack, the contract should implement a guard condition in the `doLiquidation` function to ensure that it is not called again while it is already executing. This can be achieved by using the `require` statement with a mutex variable that is set to true before calling external contract functions and set to false after the external calls have completed. This will ensure that the `doLiquidation` function is not reentered while it is already executing.

## 4. Incorrect assignment in constructor

Summary: 
The `token0IsUnderlying` variable is being set incorrectly in the constructor of the PaprController contract. The assignment `token0IsUnderlying = address(underlying) < address(papr);` is using the address of the `underlying` contract as the left operand and the address of the `papr` contract as the right operand. However, the `papr` contract is not defined or initialized in the constructor, leading to an undefined reference error.

Impact: 
This issue could lead to incorrect behavior of the contract and potentially cause unintended consequences for users of the contract.

Recommendation: 
To fix this issue, the `papr` contract should be defined and initialized in the constructor before it is used in the assignment for the `token0IsUnderlying` variable. Alternatively, a different approach can be used to correctly set the value of the `token0IsUnderlying` variable.

## 5. Incorrect value of constant BIPS_ONE in PaprController contract

Summary:
The value of the constant BIPS_ONE in the PaprController contract is incorrectly set to 1e4 (10,000). This value should be 1e2 (100) to represent 100% in basis points (bips).

Impact:
The incorrect value of BIPS_ONE will affect any calculations or logic that rely on this constant, potentially leading to incorrect results and unintended behavior.

Recommendation:
To fix this issue, the value of BIPS_ONE should be changed from 1e4 to 1e2. It is recommended to thoroughly test the contract after making this change to ensure that it does not cause any unintended consequences.

## 6. Lack of Validation in Constructor

Summary: 
The constructor of the PaprController contract does not properly validate the input arguments, specifically the `indexMarkRatioMax` and `indexMarkRatioMin` parameters. This can lead to vulnerabilities in the contract and potentially allow an attacker to manipulate the behavior of the contract.

Impact: 
The lack of validation in the constructor could allow an attacker to manipulate the behavior of the contract, potentially leading to unintended or malicious consequences.

Recommendation: 
It is recommended to properly validate the input arguments in the constructor, ensuring that the `indexMarkRatioMax` and `indexMarkRatioMin` parameters are within the expected range and do not allow for unexpected or malicious behavior. This can be done by adding checks to ensure that these parameters are within the desired range and throwing an exception if they are not.

## 7. Reentrancy vulnerability in `addCollateral` function

Summary: 
The `addCollateral` function in the contract does not properly check for reentrancy attacks and is therefore vulnerable to such attacks.

Impact: 
An attacker could potentially exploit this vulnerability to drain funds from the contract and/or execute unintended contract functions.

Recommendation: 
The `addCollateral` function should be updated to include a guard condition to check for reentrancy attacks before executing any contract functions that modify state or send funds. This can be done by using a mutex, by checking the `block.number` or `block.timestamp` within the function, or by using the `ReentrancyGuard` contract from OpenZeppelin.

## 8. Lack of input validation on collateralOwner mapping

Summary:
The `collateralOwner` mapping in the `PaprController` contract is not properly validated for input, which could allow an attacker to set the value of this mapping to any address they choose. This can potentially allow an attacker to take control of collateral that they do not own.

Impact:
This vulnerability could allow an attacker to take control of collateral that they do not own, which could lead to the attacker being able to liquidate the collateral and potentially profit from it. This could lead to financial loss for the users of the contract.

Recommendation:
To fix this vulnerability, input validation should be implemented for the `collateralOwner` mapping. This could involve checking that the provided address is the actual owner of the collateral before allowing it to be added to the mapping.

## 9. Incorrect implementation of `liquidationPenaltyBips` variable

Summary: 
The `liquidationPenaltyBips` variable in the `PaprController` contract is incorrectly implemented as a constant and is set to 1000. This value is meant to represent 10% as a basis point value, but 1000 basis points is equivalent to 100%. This will cause the liquidation penalty calculation to be incorrect and result in an incorrect penalty being applied.

Impact: 
This bug will result in incorrect liquidation penalty calculations, which can have significant financial implications for users of the contract.

Recommendation: 
To fix this issue, the `liquidationPenaltyBips` variable should be set to the correct value of 1000 basis points, or 10%. This will ensure that the correct liquidation penalty is applied. Additionally, it is recommended to thoroughly test the contract to ensure that the bug has been properly fixed and that all calculations are being performed correctly.

## 10. Potential reentrancy vulnerability in PaprController contract

Summary:
The `PaprController` contract contains a potential reentrancy vulnerability when calling the `withdraw()` function. The `withdraw()` function calls the `safeTransferFrom()` function of the ERC721 contract to transfer ownership of a collateral token to the caller. However, the `safeTransferFrom()` function can execute arbitrary code when it calls the `onERC721Received()` function on the recipient contract, which can potentially call the `withdraw()` function again, leading to a reentrancy attack.

Impact:
If exploited, an attacker could potentially exploit this vulnerability to repeatedly call the `withdraw()` function and drain the contract of its collateral tokens. This could result in financial loss for the contract owner and users of the contract.

Recommendation:
To prevent this vulnerability, the `withdraw()` function should be modified to use a reentrancy guard or call the `safeTransferFrom()` function in a `require()` statement that checks the current balance of the contract. This will ensure that the `withdraw()` function can only be called once per execution and prevent a reentrancy attack.

## 11. Incorrectly formatted Solidity contract import

Summary: 
The import statement for the `Multicallable` contract is using a different syntax than the other import statements in the code. The `solady/utils/Multicallable.sol` import should be changed to `solmate/utils/Multicallable.sol` to match the syntax of the other import statements.

Impact: 
This incorrect import syntax will cause a compile-time error when trying to deploy the contract.

Recommendation: 
Update the import statement for the `Multicallable` contract to match the syntax of the other import statements in the code. The correct import statement should be `import {Multicallable} from "solmate/utils/Multicallable.sol";`.

## 12. Missing check on `isAllowed` variable before calling `createVault` function

Summary:
The `createVault` function in the `PaprController` contract allows anyone to create a vault, even if they are not marked as `isAllowed` in the `isAllowed` mapping. This means that any user can create a vault without the proper authorization.

Impact:
This vulnerability allows unauthorized users to create vaults, which could potentially lead to the loss of funds.

Recommendation:
To fix this issue, the `createVault` function should include a check on the `isAllowed` mapping to ensure that only authorized users are able to create vaults. This check should be added before any other functionality in the `createVault` function is executed.

Example fix:
```
function createVault(ERC721 collateral, uint256 value) external override {
    require(isAllowed[msg.sender], "Unauthorized user");
    // rest of the function
}
```

## 13.  Incorrect maxLTV value in constructor

Summary:
The constructor for the PaprController contract has an issue where the `maxLTV` value is not correctly set. The issue is caused by a missing assignment statement for the `maxLTV` variable.

Impact:
The incorrect value of `maxLTV` could lead to unintended behavior in the contract, potentially causing issues with the correct functioning of the contract.

Recommendation:
To fix this issue, add an assignment statement for the `maxLTV` variable in the constructor, setting its value to the `_maxLTV` parameter passed in to the constructor. This will ensure that the correct value is assigned to `maxLTV` and the contract functions as intended.
Example fix:
```
constructor(
        string memory name,
        string memory symbol,
        uint256 _maxLTV,
        uint256 indexMarkRatioMax,
        uint256 indexMarkRatioMin,
        ERC20 underlying,
        address oracleSigner
    )
        NFTEDAStarterIncentive(1e17)
        UniswapOracleFundingRateController(underlying, new PaprToken(name, symbol), indexMarkRatioMax, indexMarkRatioMin)
        ReservoirOracleUnderwriter(oracleSigner, address(underlying))
    {
        maxLTV = _maxLTV; // add this line to fix the issue
        token0IsUnderlying = address(underlying) < address(papr);
        uint256 underlyingONE = 10 ** underlying.decimals();
        uint160 initSqrtRatio;

        // initialize the pool at 1:1
        if (token0IsUnderlying) {
            initSqrtRatio = 1;
        } else {
            initSqrtRatio = 1e18 / underlyingONE;
        }
        fundUnderlyingFromOracle(initSqrtRatio * underlyingONE);
        fundPaprFromOracle(initSqrtRatio * 1e18);
    }
```
# 6th report for : NFTEDA.sol https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol

## 1. Insufficient Payment validation not correctly implemented

Summary:
The function `_purchaseNFT` in the NFTEDA contract is supposed to check that the caller has sent the correct amount of the payment asset to the contract before transferring the NFT to the buyer. However, the `_purchaseNFT` function does not check the amount of payment received, and only checks that the maxPrice paid by the buyer is greater than or equal to the current auction price. This means that if a buyer sends any amount of the payment asset, they can purchase the NFT for any price less than or equal to their maxPrice, even if the payment asset amount is less than the current auction price.

Impact:
This vulnerability allows an attacker to purchase an NFT for a lower price than intended by the seller. This can result in a financial loss for the seller and can also undermine the integrity of the auction process.

Recommendation:
To fix this vulnerability, the `_purchaseNFT` function should check that the caller has sent the correct amount of the payment asset before transferring the NFT. This can be done by adding a check that the amount of the payment asset received by the contract is equal to the current auction price. If the payment asset amount is less than the current auction price, the function should revert with an `InsufficientPayment` error. The error should include the received and expected payment amounts to allow the caller to understand why the purchase has failed.

## 2. InvalidAuction error when calling auctionCurrentPrice with an invalid auction

Summary:
The contract function `auctionCurrentPrice` calls `auctionStartTime` and reverts if the returned value is 0. However, there is no mechanism to check if the passed `auction` is a valid auction. As a result, if an attacker passes an invalid `auction` to the function, it will always revert with the `InvalidAuction` error, leading to a loss of availability for the contract.

Impact:
This vulnerability allows an attacker to disrupt the availability of the contract by repeatedly calling `auctionCurrentPrice` with invalid auctions. This can potentially lead to a denial of service attack on the contract.

Recommendation:
To fix this vulnerability, it is recommended to add a mechanism to validate the passed `auction` before calling `auctionStartTime`. This can be done by storing a mapping of valid auctions and checking if the passed `auction` is in the mapping before calling `auctionStartTime`.

# 7th report for : PoolAddress.sol https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol

## 1. Incorrect sorting of PoolKey tokens

Summary:
The function `getPoolKey` in the `PoolAddress` library is intended to return a `PoolKey` struct with the tokens sorted in ascending order. However, the current implementation does not correctly sort the tokens and instead relies on the assumption that `tokenA` is always less than `tokenB`. This could lead to incorrect results if `tokenB` is actually less than `tokenA`.

Impact:
This issue could potentially lead to incorrect pool addresses being computed, depending on the input values of `tokenA` and `tokenB`. This could result in unexpected behavior and potentially result in loss of funds for users of the pool.

Recommendation:
To fix this issue, the `getPoolKey` function should properly sort the tokens before returning the `PoolKey` struct. This can be done by adding a comparison statement before assigning the values to the struct, such as `if (tokenA > tokenB) (tokenA, tokenB) = (tokenB, tokenA);`. This will ensure that the `PoolKey` struct always contains the tokens in ascending order, as intended.

# 8th report for : OracleLibrary.sol https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol

## 1. Uninitialized variable tick in getQuoteAtTick function

Summary:
The `tick` variable in the `getQuoteAtTick` function is not initialized with a value before it is used. This can cause unpredictable behavior and may lead to security vulnerabilities.

Impact:
The use of an uninitialized variable can result in incorrect calculations and potentially harmful actions being taken based on those calculations. This can result in loss of funds or other unintended consequences for users of the contract.

Recommendation:
Initialize the `tick` variable with a value before using it in the function. It is also recommended to add appropriate safeguards to ensure that the value of `tick` is within an acceptable range before using it in calculations.

## 2. Time Weighted Average Tick (TWAT) calculation can result in incorrect values

Summary:
The function timeWeightedAverageTick() in the OracleLibrary library can produce incorrect results due to a potential integer overflow issue. The function calculates the TWAT by dividing the difference between two integer values, startTick and endTick, by the integer value twapDuration. However, if the difference between startTick and endTick is negative, and the result of the modulo operation (delta % twapDuration) is not equal to zero, the function decrements the TWAT value by 1. This could lead to incorrect TWAT values being returned if the difference between startTick and endTick is negative and the result of the modulo operation is not equal to zero.

Impact:
Incorrect TWAT values could potentially lead to incorrect calculations and incorrect results in other functions or contracts that rely on the TWAT calculation.

Recommendation:
To fix this issue, the function should be modified to handle negative values of delta correctly by adding a separate check for negative values before performing the modulo operation. This would ensure that the TWAT value is correctly calculated for all cases, regardless of the sign of delta.

## 3.  Integer Overflow Vulnerability in OracleLibrary.getQuoteAtTick()

Summary:
The `getQuoteAtTick()` function in the OracleLibrary contract contains an integer overflow vulnerability that could potentially be exploited by an attacker. The vulnerability exists in the `if (sqrtRatioX96 <= type(uint128).max)` check, which is used to determine whether a higher precision calculation should be used. However, this check is not sufficient to prevent an integer overflow when `sqrtRatioX96` is larger than the maximum value of a `uint128`.

Impact:
If an attacker is able to manipulate the inputs to the `getQuoteAtTick()` function such that `sqrtRatioX96` is larger than the maximum value of a `uint128`, they could potentially trigger an integer overflow, leading to incorrect and potentially maliciously manipulated results.

Recommendation:
To fix this vulnerability, the `if (sqrtRatioX96 <= type(uint128).max)` check should be replaced with a more robust check that properly handles the case where `sqrtRatioX96` is larger than the maximum value of a `uint128`. One possible solution would be to use a `require()` statement to ensure that `sqrtRatioX96` is within the acceptable range before performing the calculation.

# 9th report for : UniswapHelpers.sol https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol

## 1. Revert on TooLittleOut when amountOut is less than minOut

Summary: 
The function `swap` in the `UniswapHelpers` library reverts when the amount of token0 or token1 received is less than the minimum specified amount of token0 or token1 to receive. This occurs when the `amountOut` variable is less than the `minOut` parameter in the `if (amountOut < minOut)` statement.

Impact: 
When this issue occurs, the swap will not be executed and the user will not receive the minimum specified amount of token0 or token1. This can result in a loss of funds for the user.

Recommendation: 
To fix this issue, the `if (amountOut < minOut)` statement should be removed or modified to allow the swap to be executed even if the amount of token0 or token1 received is less than the minimum specified amount. Alternatively, the `minOut` parameter could be removed from the function, and the user can specify the minimum amount of token0 or token1 to receive in the `amountSpecified` parameter.
