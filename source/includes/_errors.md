# Errors

API errors typically include the errors attribute in graphql response payload, following are typical http error codes the integration could receive.

Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is invalid, typically caused due to invalt eid graphql parameters of .
401 | Unauthorized -- Your API key is wrong.
429 | Too Many Requests -- Rate limit exceeded.
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.
