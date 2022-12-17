## Summary<a name="Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW&#x2011;1) | Use `_safeMint` instead of `_mint` | 1 |
| [LOW&#x2011;2](#LOW&#x2011;2) | Contracts are not using their OZ Upgradeable counterparts | 1 |
| [LOW&#x2011;3](#LOW&#x2011;3) | Missing parameter validation in `constructor` | 2 |
| [LOW&#x2011;4](#LOW&#x2011;4) | Prevent division by 0 | 1 |
| [LOW&#x2011;5](#LOW&#x2011;5) | `ecrecover` may return empty address | 1 |
| [LOW&#x2011;6](#LOW&#x2011;6) | Use of `ecrecover` is susceptible to signature malleability | 1 |

Total: 7 contexts over 6 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC&#x2011;1) | Critical Changes Should Use Two-step Procedure | 4 |
| [NC&#x2011;2](#NC&#x2011;2) | Use a more recent version of Solidity | 10 |
| [NC&#x2011;3](#NC&#x2011;3) | Missing event for critical parameter change | 1 |
| [NC&#x2011;4](#NC&#x2011;4) | Implementation contract may not be initialized | 5 |
| [NC&#x2011;5](#NC&#x2011;5) | Large multiples of ten should use scientific notation | 1 |
| [NC&#x2011;6](#NC&#x2011;6) | Use of Block.Timestamp | 5 |
| [NC&#x2011;7](#NC&#x2011;7) | Use `bytes.concat()` | 2 |
| [NC&#x2011;8](#NC&#x2011;8) | Use `delete` to Clear Variables | 2 |
| [NC&#x2011;9](#NC&#x2011;9) | Use Underscores for Number Literals  | 1 |
| [NC&#x2011;10](#NC&#x2011;10) | No full coverage of in scope files  | 1 |

Total: 41 contexts over 10 issues

## Low Risk Issues



### <a href="#Summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Use `_safeMint` instead of `_mint`

According to openzepplin's ERC721, the use of `_mint` is discouraged, use _safeMint whenever possible.
https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-_mint-address-uint256-

#### <ins>Proof Of Concept</ins>


```solidity
25: _mint(to, amount);
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L25



#### <ins>Recommended Mitigation Steps</ins>

Use `_safeMint` whenever possible instead of `_mint`



### <a href="#Summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Contracts are not using their OZ Upgradeable counterparts

The non-upgradeable standard version of OpenZeppelin’s library are inherited / used by the contracts.
It would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

#### <ins>Proof of Concept</ins>

```solidity
9: import {Ownable2Step} from "openzeppelin-contracts/access/Ownable2Step.sol";

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L9



#### <ins>Recommended Mitigation Steps</ins>

Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.



### <a href="#Summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Missing parameter validation in `constructor`

Some parameters of constructors are not checked for invalid values.

#### <ins>Proof Of Concept</ins>

```solidity
51: address _oracleSigner
51: address _quoteCurrency

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L51-L51



#### <ins>Recommended Mitigation Steps</ins>

Validate the parameters.



### <a href="#Summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Prevent division by 0

On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code.

These functions can be called with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

#### <ins>Proof Of Concept</ins>


```solidity
558: return maxLoanUnderlying / cachedTarget
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L558




#### <ins>Recommended Mitigation Steps</ins>

Recommend making sure division by 0 won’t occur by checking the variables beforehand and handling this edge case.



### <a href="#Summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> `ecrecover` may return empty address

There is a common issue that ecrecover returns empty (0x0) address when the signature is invalid. function recoverAddrImpl should check that before returning the result of ecrecover.

While unlikely to occur due to the value of `oracleSigner` being set in the constructor, it is still best practice to check for address(0x0) if `oracleSigner` was accidentally set as address(0x0) in addition that there is no address(0x0) checks in the constructor.

#### <ins>Proof Of Concept</ins>


```solidity
64: function underwritePriceForCollateral(ERC721 asset, PriceKind priceKind, OracleInfo memory oracleInfo)
        public
        returns (uint256)
    {
        address signerAddress = ecrecover(
            keccak256(
                abi.encodePacked(
                    "\x19Ethereum Signed Message:\n32",
                    // EIP-712 structured-data hash
                    keccak256(
                        abi.encode(
                            keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),
                            oracleInfo.message.id,
                            keccak256(oracleInfo.message.payload),
                            oracleInfo.message.timestamp
                        )
                    )
                )
            ),
            oracleInfo.sig.v,
            oracleInfo.sig.r,
            oracleInfo.sig.s
        );
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L68



#### <ins>Recommended Mitigation Steps</ins>

See the solution here: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v3.4.0/contracts/cryptography/ECDSA.sol#L68



### <a href="#Summary">[LOW&#x2011;6]</a><a name="LOW&#x2011;6"> Use of `ecrecover` is susceptible to signature malleability

The built-in EVM precompile `ecrecover` is susceptible to signature malleability, which could lead to replay attacks.
References:  https://swcregistry.io/docs/SWC-117,  https://swcregistry.io/docs/SWC-121, and  https://medium.com/cryptronics/signature-replay-vulnerabilities-in-smart-contracts-3b6f7596df57.
While this is not immediately exploitable, this may become a vulnerability if used elsewhere.

#### <ins>Proof Of Concept</ins>


```solidity
64: function underwritePriceForCollateral(ERC721 asset, PriceKind priceKind, OracleInfo memory oracleInfo)
        public
        returns (uint256)
    {
        address signerAddress = ecrecover(
            keccak256(
                abi.encodePacked(
                    "\x19Ethereum Signed Message:\n32",
                    // EIP-712 structured-data hash
                    keccak256(
                        abi.encode(
                            keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)"),
                            oracleInfo.message.id,
                            keccak256(oracleInfo.message.payload),
                            oracleInfo.message.timestamp
                        )
                    )
                )
            ),
            oracleInfo.sig.v,
            oracleInfo.sig.r,
            oracleInfo.sig.s
        );
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L68



#### Recommended Mitigation Steps
Consider using OpenZeppelin’s ECDSA library (which prevents this malleability) instead of the built-in function.
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/cryptography/ECDSA.sol#L138-L149



## Non Critical Issues

### <a href="#Summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Critical Changes Should Use Two-step Procedure

The critical procedures should be two step process.

See similar findings in previous Code4rena contests for reference:
https://code4rena.com/reports/2022-06-illuminate/#2-critical-changes-should-use-two-step-procedure

#### <ins>Proof Of Concept</ins>

```solidity
350: function setPool(address _pool) external override onlyOwner {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L350

```solidity
355: function setFundingPeriod(uint256 _fundingPeriod) external override onlyOwner {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L355

```solidity
360: function setLiquidationsLocked(bool locked) external override onlyOwner {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L360

```solidity
365: function setAllowedCollateral(IPaprController.CollateralAllowedConfig[] calldata collateralConfigs)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L365



#### <ins>Recommended Mitigation Steps</ins>

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.



### <a href="#Summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Use a more recent version of Solidity

<a href="https://blog.soliditylang.org/2021/04/21/solidity-0.8.4-release-announcement/">0.8.4</a>:
bytes.concat() instead of abi.encodePacked(<bytes>,<bytes>)

<a href="https://blog.soliditylang.org/2022/02/16/solidity-0.8.12-release-announcement/">0.8.12</a>: 
string.concat() instead of abi.encodePacked(<str>,<str>)

<a href="https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/">0.8.13</a>: 
Ability to use using for with a list of free functions

<a href="https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/">0.8.14</a>:

ABI Encoder: When ABI-encoding values from calldata that contain nested arrays, correctly validate the nested array length against calldatasize() in all cases.
Override Checker: Allow changing data location for parameters only when overriding external functions.

<a href="https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/">0.8.15</a>:

Code Generation: Avoid writing dirty bytes to storage when copying bytes arrays.
Yul Optimizer: Keep all memory side-effects of inline assembly blocks.

<a href="https://blog.soliditylang.org/2022/08/08/solidity-0.8.16-release-announcement/">0.8.16</a>:

Code Generation: Fix data corruption that affected ABI-encoding of calldata values represented by tuples: structs at any nesting level; argument lists of external functions, events and errors; return value lists of external functions. The 32 leading bytes of the first dynamically-encoded value in the tuple would get zeroed when the last component contained a statically-encoded array.

<a href="https://blog.soliditylang.org/2022/09/08/solidity-0.8.17-release-announcement/">0.8.17</a>:

Yul Optimizer: Prevent the incorrect removal of storage writes before calls to Yul functions that conditionally terminate the external EVM call.

#### <ins>Proof Of Concept</ins>


```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IFundingRateController.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IPaprController.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/interfaces/IUniswapOracleFundingRateController.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L4

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/UniswapHelpers.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/interfaces/INFTEDA.sol#L2

```solidity
pragma solidity >=0.8.0;
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/libraries/EDAPrice.sol#L2



#### <ins>Recommended Mitigation Steps</ins>

Consider updating to a more recent solidity version.






### <a href="#Summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Missing event for critical parameter change

When changing state variables events are not emitted. Emitting events allows monitoring activities with off-chain monitoring tools.

#### <ins>Proof Of Concept</ins>


```solidity
360: function setLiquidationsLocked(bool locked) external override onlyOwner {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L360







### <a href="#Summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Implementation contract may not be initialized

OpenZeppelin recommends that the initializer modifier be applied to constructors. 
Per OZs Post implementation contract should be initialized to avoid potential griefs or exploits.
https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/5

#### <ins>Proof Of Concept</ins>


```solidity
67: constructor(
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
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L67

```solidity
18: constructor(string memory name, string memory symbol)
        ERC20(string.concat("papr ", name), string.concat("papr", symbol), 18)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L18

```solidity
51: constructor(address _oracleSigner, address _quoteCurrency)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L51

```solidity
34: constructor(ERC20 _underlying, ERC20 _papr, uint256 _targetMarkRatioMax, uint256 _targetMarkRatioMin)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L34

```solidity
33: constructor(uint256 _auctionCreatorDiscountPercentWad)
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/extensions/NFTEDAStarterIncentive.sol#L33





### <a href="#Summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Large multiples of ten should use scientific notation

Use (e.g. 1e6) rather than decimal literals (e.g. 100000), for better code readability.

#### <ins>Proof Of Concept</ins>


```solidity
92: address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L92






### <a href="#Summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Use of Block.Timestamp

Block timestamps have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are time-dependent. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.
References: SWC ID: 116

#### <ins>Proof Of Concept</ins>


```solidity
321: if (block.timestamp - info.latestAuctionStartTime < liquidationAuctionMinSpacing) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L321

```solidity
394: if (_lastUpdated == block.timestamp) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L394

```solidity
46: if (_lastUpdated == block.timestamp) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L46

```solidity
64: if (_lastUpdated == block.timestamp) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L64

```solidity
73: if (_lastUpdated == block.timestamp) {
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/UniswapOracleFundingRateController.sol#L73



#### <ins>Recommended Mitigation Steps</ins>
Block timestamps should not be used for entropy or generating random numbers—i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking), completing an ICO after a few weeks, or enforcing expiry dates. It is sometimes recommended to use block.number and an average block time to estimate times; with a 10 second block time, 1 week equates to approximately, 60480 blocks. Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.





### <a href="#Summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Use `bytes.concat()`

Solidity version 0.8.4 introduces `bytes.concat()` (vs `abi.encodePacked(<bytes>,<bytes>)`)

#### <ins>Proof Of Concept</ins>


```solidity
70: abi.encodePacked(
                    "/x19Ethereum Signed Message:/n32",
                    // EIP-712 structured-data hash
                    keccak256(
                        abi.encode(
                            keccak256("Message(bytes32 id,bytes payload,uint256 timestamp)

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/ReservoirOracleUnderwriter.sol#L70

```solidity
37: abi.encodePacked(
                            hex"ff",
                            factory,
                            keccak256(abi.encode(key.token0, key.token1, key.fee)

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/PoolAddress.sol#L37




#### <ins>Recommended Mitigation Steps</ins>

Use `bytes.concat()` and upgrade to at least Solidity version 0.8.4 if required. 



### <a href="#Summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Use `delete` to Clear Variables

`delete a` assigns the initial value for the type to `a`. i.e. for integers it is equivalent to `a = 0`, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. Similarly, it can also be used to set an address to zero address. It has no effect on whole mappings though (as the keys of mappings may be arbitrary and are generally unknown). However, individual keys and what they map to can be deleted: If `a` is a mapping, then `delete a[x]` will delete the value stored at `x`.

The `delete` key better conveys the intention and is also more idiomatic. Consider replacing assignments of zero with `delete` statements.

#### <ins>Proof Of Concept</ins>

```solidity
526: _vaultInfo[auction.nftOwner][auction.auctionAssetContract].latestAuctionStartTime = 0;

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L526

```solidity
58: secondAgos[0] = 0;

```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/libraries/OracleLibrary.sol#L58





### <a href="#Summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Use Underscores for Number Literals

#### <ins>Proof Of Concept</ins>


```solidity
92: address _pool = UniswapHelpers.deployAndInitPool(address(underlying), address(papr), 10000, initSqrtRatio);
```

https://github.com/with-backed/papr/tree/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L92



### <a href="#Summary">[NC&#x2011;10]</a><a name="NC&#x2011;10">  No full coverage of in scope files

It is recommended to have full coverage on all in scope files of the project.

#### <ins>Proof Of Concept</ins>

As stated in the scope of the project:

| |SLOC|Coverage|
|-|:-|:-:|
| Total (over 14 files): | 1043  |  83.33% |
