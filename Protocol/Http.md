# HTTP headers
## Authentication
## Caching
* Expires
	* The Expires header contains the date/time after which the response is considered stale.
	* If there is a Cache-Control header with the max-age or s-maxage directive in the response, the Expires header is ignored.
* Cache-Control
	* The Cache-Control HTTP header holds directives (instructions) for caching in both requests and responses. A given directive in a request does not mean the same directive should be in the response.
### Cacheability
* A response is normally cached by the browser if:
	* It has a status code of 301, 302, 307, 308, or 410 and Cache-Control does not have no-store, or if proxy, does not have private and Authorization is unset.
	* Either
		* has a status code of 301, 302, 307, 308, or 410 or
		* has public, max-age or s-maxage in Cache-Control or
		* has Expires set

## Conditionals
* Last-Modified
    * The Last-Modified response HTTP header contains the date and time at which the origin server believes the resource was last modified. 
    * It is used as a validator to determine if a resource received or stored is the same. 
    * Less accurate than an ETag header, it is a fallback mechanism. Conditional requests containing If-Modified-Since or If-Unmodified-Since headers make use of this field.
* ETag
    * It is a validator, a unique string identifying the version of the resource. Conditional requests using If-Match and If-None-Match use this value to change the behavior of the request.
    
## Connection management
## Content negotiation
## CORS
## Message body information
* Content-Encoding
### Compressing with gzip
* The Accept-Encoding header is used for negotiating content encoding.
* The server responds with the scheme used, indicated by the Content-Encoding response header.
### Compressing through Application/Container/Reverse Proxy
* TBD...
## Security
## Transfer coding
# References
* [循序漸進理解 HTTP Cache 機制](https://blog.techbridge.cc/2017/06/17/cache-introduction/)
