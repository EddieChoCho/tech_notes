## REST

### Resource
#### URI
#### Uniform interface
* HTTP methods
|HTTP method|Description|Details                                 |
|-----------|-----------|----------------------------------------|
|GET        |Get a representation of the target resource’s state.|No side effect, Idempotent, Cacheable|
|PUT        |Set the target resource’s state to the state defined by the representation enclosed in the request.|Idempotent|
|DELETE.    |Delete the target resource’s state.|Idempotent|
|HEAD       ||Safe, Idempotent|
|POST       |Let the target resource process the representation enclosed in the request.||

### Protocol
#### Client–server
* HTTP Intermediaries can be added at various points in the request-response path.
* Intermediaries include proxies and gateways. (Proxies are chosen by the clients. Gateways are chosen by the origin servers.)

#### Statelessness
* Each request is independent from the others allowing intermediaries to work on a singel intraction without knowing the entire topology.
* Since different requests may travel through different intermediaries. There may be no chance of visibility between the intractions.
#### Cacheable
#### Layered system

### Benefits
#### Network Performance
* Efficiency
    * All of those caches help along the way. The requests may not have to reach all the way back to the origin server.
    * Control data allows the signaling of compression. A response can be gzipped before sent to the client.

* Scalability
    * Gateways allows you to distribute traffic among a large set of servers based on method, URI, content type or any other headers coming from the request.
    * Caching helps reduce the actual number of requests that make it all the way back to the servers.
    * Statelessness allows requests to be routed through different gateways and proxies thus avoiding introducing bottlenecks.
    * Statelessness allowing more intermediaries to be added as needed.
        
* User Perceived Performance
    * User Perceived Performance is increased by having a reduced set of known media types that allows browsers to handle known types much faster.
    	e.g. partial rendering of HTML documents as they download.
    * Code on demand allows computations to be moved closer to the client or closer to the server depending on where the work can be done fastest.

## Further Reading
* [RFC 2616](http://www.ietf.org/rfc/rfc2616.txt​)
* [RFC 3986](https://www.rfc-editor.org/rfc/rfc3986.txt)
* [Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)
* [Caching Tutorial](http://www.mnot.net/cache_docs/)
* [Nginx ngx_http_gzip_module module](http://nginx.org/en/docs/http/ngx_http_gzip_module.html)
* [BREACH attack0](https://en.wikipedia.org/wiki/BREACH)
* [Principles & Best practices of REST API Design](https://blog.devgenius.io/best-practice-and-cheat-sheet-for-rest-api-design-6a6e12dfa89f)

## References
* [Intro to REST - Google Developers](https://youtu.be/YCcAE2SCQ6k)
