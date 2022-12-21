## [Low-1] `oracleSigner` is immutable

If the keys of the `oracleSigner` are lost, then the market will be frozen as the signed message requires to be "fresh" (signed at a timestamp that does not exceed [20 minutes](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L38)).
It's a good feature for transparency to have `oracleSigner` as an immutable variable, but in some cases it can be really dangerous.
There is no need to recall past hacks due to bad OPSEC, but losing the private key, sharing it by error (phishing, hacking ...) had some pretty bad consequences.

A solution would be to make it a storage variable and being able to change it from a multisig and a timelock to mitigate those types of issues.

## [Low-2] The oracle can be the victim of a signature replay attack

`underwritePriceForCollateral()` is not EIP712 compliant. It should pass the `DOMAIN_SEPARATOR()` in order to avoid this issue.
This is written in the `EIP-712` specification:

"The way to solve this (*replay attacks*) is by introducing a domain separator, a 256-bit number. This is a value unique to each domain that is ‘mixed in’ the signature. It makes signatures from different domains incompatible. The domain separator is designed to include bits of DApp unique information such as the name of the DApp, the intended validator contract address, the expected DApp domain name, etc. The user and user-agent can use this information to mitigate phishing attacks, where a malicious DApp tries to trick the user into signing a message for another DApp."

https://eips.ethereum.org/EIPS/eip-712

As from the whitepaper, the oracle is pushing the current prices using signed messages and took ["TrustUs"](https://github.com/ZeframLou/trustus/blob/main/src/Trustus.sol) as a reference point which included this domain separator.

## [NC-1] Incorrect error in `underwritePriceForCollateral()`

```solidity
if (
    oracleInfo.message.timestamp > block.timestamp || oracleInfo.message.timestamp + VALID_FOR < block.timestamp
) {
    revert OracleMessageTooOld();
}
```

The error that is returned when the price from the oracle is too "young" or too "old" seems to be incorrect. A quick fix would be to change it to:

```solidity
if (oracleInfo.message.timestamp > block.timestamp) {
    revert OracleMessageTooYoung();
} else if (oracleInfo.message.timestamp + VALID_FOR < block.timestamp) {
    revert OracleMessageTooOld();
}
```

## [NC-2] Misleading information in the NATSPEC comment

This one is related to my other finding "There is no way to extract fees when someones want to reduce a debt by paying with underlying tokens".
In the [`IPaprController`](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L164), it is explained that the `buyAndReduceDebt` removes debt from a vault while allowing to pay with the controllers underlying tokens, while the tokens should clearly go from the payer to the Uniswap pool in order to buy some `PAPR`.
Invalid concern if the meaning was that the underlying were concerning the **address** of the underlying that is accepted to be used in order to repay the debt of a vault.

## [NC-3] Typo

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L150
`_lastUpdates` should be replaced with `_lastUpdated`.