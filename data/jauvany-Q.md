Vulnerability: Use of externally-owned addresses as controller in IFundingRateController.sol

Vulnerability Details
IFundingRateController.sol contract uses externally-owned addresses as the controller

Impact
It is advised against using addresses that are externally-owned as the controller since the contract can be manipulated or tampered with.

Recommended Mitigation Steps
The controller should not be externally-owned
