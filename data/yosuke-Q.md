## [YO NC-1] No need for the second argument of the onERC721Received() function

### Handle
yosuke

## Vulnerability details
### Impact

### Proof of Concept
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L159

```solidity=
function onERC721Received(address from, address, uint256 _id, bytes calldata data) 
        external
        override
        returns (bytes4)
    {}
```

### Recommended Mitigation Steps
Remove the second argument of the onERC721Received() function.

after
```solidity=
function onERC721Received(address from, uint256 _id, bytes calldata data) 
        external
        override
        returns (bytes4)
    {}
```