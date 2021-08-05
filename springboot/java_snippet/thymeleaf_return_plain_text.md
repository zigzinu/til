# How to return plain text instead of template when using Thymeleaf?

Some controller functions might want to return some value instead of redirecting to a html template. 

### Solution is to add `@ResponseBody` tag to the controller function.

```
	@GetMapping("/exitSalePercentage")
	@ResponseBody
	public String getExitSalePercentage(@RequestParam(required=true) String productId) throws Exception {
		
		String percentage = "" + service.queryEntrySalePercentage(productId);
		LOGGER.info("Entry Sale [product: {} percentage: {}%]", productId, percentage);
		return percentage;
	}
```
