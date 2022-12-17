**src/UniswapOracleFundingRateController.sol**
- L12 - The IFundingRateController interface is imported, but it is never used, therefore it should be removed.


**src/PaprController.sol**
- L41/44/47/50/54 - Multiple constants are created in storage that are immutable, they should be constants since that is what they really are. In addition, the names of the variables should be in upper case and not in lower case.


**src/NFTEDA/NFTEDA.sol**
- L4 - The ERC721 class is imported, but it is never used, therefore it should be removed.

- L18 - The InsufficientPayment error is created, but it is never used, therefore it should be eliminated.


**src/libraries/UniswapHelpers.sol**
- L7 - The FullMath class is imported, but it is never used, therefore it should be removed.

- L111 - A division is made by an input (token0ONE) therefore it should be validated before if it is != 0, since an exception should be thrown if this happens.


**src/interfaces/IFundingRateController.sol**
- L4/47/51 - An abstract class (solmate/tokens/ERC20.sol) is imported and used to return some functions, when it would be less expensive to return an IERC20.


**src/NFTEDA/interfaces/INFTEDA.sol**
- L4/5/16/25/40/45 - It is imported and used to return some abstract class functions (solmate/tokens/ERC20.sol and solmate/tokens/ERC721.sol), when it would be less expensive to return an IERC20 and an IERC721.


**src/interfaces/IPaprController.sol**
- L5/6/13/67/73/79/85/139/148/159/169/231/272 - It is imported and used to rotate in some abstract class functions (solmate/tokens/ERC20.sol and solmate/ tokens/ERC721.sol), when it would be less expensive to return an IERC20 and an IERC721.
