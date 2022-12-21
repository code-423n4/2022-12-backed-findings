## missing address 0 check
function is missing address 0 check consider adding one
[PaprController.sol#L109-L134](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L109-L134)
[PaprController.sol#L138-L145](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L138-L145)

## Use underscores for number literals
[PaprController.sol#L54](https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L54)
Description:
There is occasions where certain number have been hardcoded, either in variable or in the code itself. Large numbers can become hard to read.

Recommendation:
Consider using underscores for number literals to improve its readability.