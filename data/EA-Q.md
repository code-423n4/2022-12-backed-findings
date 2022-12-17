Protecting against publicly readable variables

Publicly readable auctionCreatorDiscountPercentWad and _pricePercentAfterDiscount variables:
Line of code:
	uint256 public auctionCreatorDiscountPercentWad; (line 6)
	uint256 internal _pricePercentAfterDiscount; (line 7)

Lack of function to update auctionCreatorDiscountPercentWad variable: 
This vulnerability affects the entire contract, as there is no function to update the auctionCreatorDiscountPercentWad variable.
