## L-1 There is no transfer ownership pattern

## Impact 

Papr Token's controller will not be able to transfer ownership to another address if he loses his private keys or just wanted to transfer it (ownership) to another address.

## Lines of code
- https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprToken.sol#L6

## Recommended Mitigation Steps
Inherit the Openzeppelin [```Ownable2Step```](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/access/Ownable2Step.sol) contract.