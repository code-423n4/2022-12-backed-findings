1 - Solmate's ERC20 mint() function does not check the address at the first. It may lead to token burning instead of minting:

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L25

e.g. in:

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L476
the ```mintTo``` address is not checked.

2 -  Solmate's ERC20 transferFrom() doesn't check the ```from``` and ```to``` address not to be 0x0:

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L101


