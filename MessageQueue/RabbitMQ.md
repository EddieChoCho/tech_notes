# RabbitMQ

## Connections
* The Basics
	* Since connections are meant to be long-lived, clients usually consume messages by registering a subscription and having messages delivered (pushed) to them instead of polling.
	* When a connection is no longer necessary, applications must close them to conserve resources. Apps that fail to do it run the risk of eventually exhausting its target node of resources.
	* Operating systems have a limit around how many TCP connections (sockets) a single process can have open simultaneously. 
		* The limit is often sufficient for development and some QA environments. 
		* Production environments must be configured to use a higher limit in order to support a larger number of concurrent client connections.
	* Protocol Differences: AMQP 0-9-1 v.s. AMQP 1.0
* Connection Lifecycle
	* Application configures the client library it uses to use a certain connection endpoint (e.g. hostname and port)
	* The library resolves the hostname to one or more IP addresses
	* The library opens a TCP connection to the target IP address and protocol-specific
	* After the server has accepted the TCP connection, protocol-specific negotiation procedure is performed
	* The server then authenticates the client
	* The client now can perform operations, each of which involves an authorisation check by the server.

## Channels
* Channels that can be thought of as "lightweight connections(logical connections to the broker) that share a single TCP connection".
* For applications that use multiple threads/processes for processing, it is very common to open a new channel per thread/process and not share channels between them.

### Channel Lifecycle
* Opening Channels
	* Applications open a channel right after successfully opening a connection.
	* Much like connections, channels are meant to be long lived. That is, there is no need to open a channel per operation and doing so would be very inefficient, since opening a channel is a network roundtrip.

* Closing Channels
	* When a channel is no longer needed, it should be closed. Closing a channel will render it unusable and schedule its resources to be reclaime.

* Channels and Error Handling
 	* Channels also can be closed duing to a protocol exception.

 	* Recoverable ("soft") errors in the protocol(Note: They render the channel closed but applications can open another one and try to recover or retry a number of times.):
 		* Redeclaring an existing queue or exchange with non-matching properties will fail with a 406 PRECONDITION_FAILED error
		* Accessing a resource the user is not allowed to access will fail with a 403 ACCESS_REFUSED error
		* Binding a non-existing queue or a non-existing exchange will fail with a 404 NOT_FOUND error
		* Consuming from a queue that does not exist will fail with a 404 NOT_FOUND error
		* Publishing to an exchange that does not exist will fail with a 404 NOT_FOUND error
		* Accessing an exclusive queue from a connection other than its declaring one will fail with a 405 RESOURCE_LOCKED
 	 
 	* Client libraries provide a way to observe and react to channel exceptions. 
 		* e.g., in the Java client there is a way to register an error handler and access a channel shutdown (closure) reason.
 	* Any attempted operation on a closed channel will fail with an exception.
 	* When RabbitMQ closes a channel, it notifies the client of that using an asynchronous protocol method. 
 		* In other words, an operation that caused a channel exception won't fail immediately but a channel closure event handler will fire shortly after.
 	* Some client libraries may use blocking operations that wait for a response. In this case they may communicate channel exceptions differently, e.g. using runtime exceptions, an error type, or other means appropriate for the language.

### Resource Usage
* Limiting the number of channels used per connection is highly recommended.
	* Each channel consumes a relatively small amount of memory on the client. 
		* Depending on client library's implementation detail it can also use a dedicated thread pool (or similar) where consumer operations are dispatched, and therefore one or more threads (or similar).
	* Each channel also consumes a relatively small amount of memory on the node the client is connected to, plus a few Erlang processes.

* Most applications can use a single digit number of channels per connection. 
* Those with particularly high concurrency rates (usually such applications are consumers) can start with one channel per thread/process/coroutine and switch to channel pooling when metrics suggest that the original model is no longer sustainable, e.g. because it consumes too much memory.


### Monitoring, Metrics and Diagnostics
* Detect a number of common problem
	* Channel leaks
	* High channel churn

#### Channel leaks
* A channel leak is a condition under which an application repeatedly opens channels without closing them, or at least closing only some of them.
* Channel leaks eventually exhaust the node (or multiple target nodes) of RAM and CPU resources.
* If the rate of channel open operations is consistently higher than that of channel close operations, this is evidence of a channel leak in one of the applications.
* To find out what connection leaks channels, inspect per-connection channel count.

#### High Channel Churn
* A system is said to have high channel churn when its rate of newly opened channels is consistently high and its rate of closed channels is consistently high. 
* This usually means that an application uses short lived channels or channels are often closed due to channel-level exceptions.
* While connection and disconnection rates are system-specific, rates consistently above 100/second likely indicate a suboptimal connection management by one or more applications and usually are worth investigating.

## Consumers

* The Basics
	* When a new consumer is added, assuming there are already messages ready in the queue, deliveries will start immediately.
	* An attempt to consume from a non-existent queue will result in a channel-level exception with the code of 404 Not Found and render the channel it was attempted on to be closed.

* Consumer Tags
	* The identifier that is used by client libraries to determine what handler to invoke for a given delivery.

* Consumer Lifecycle
	* Consumers are typically registered during application startup. They often would live as long as their connection or even application runs.
	* Consumers can be more dynamic and register in reaction to a system event, unsubscribing when they are no longer necessary. 
		* This is common with WebSocket clients used via Web STOMP and Web MQTT plugins, mobile clients and so on. 

* Connection Recovery
	* Client libraries offer automatic connection recovery features that involves consumer recovery.
	Usually the following recovery sequence works well:
		* Recover connection
		* Recover channels
		* Recover queues
		* Recover exchanges
		* Recover bindings
		* Recover consumers
	* In other words, consumers are usually recovered last, after their target queues and those queues' bindings are in place.

* Registering a Consumer (Subscribing, "Push API")
	* A successful subscription operation returns a subscription identifier (consumer tag). It can later be used to cancel the consumer.

* Cancelling a Consumer (Unsubscribing)
	* To cancel a consumer its identifier (consumer tag) must be known.
	* After a consumer is cancelled there will be no future deliveries dispatched to it. Note that there can still be "in flight" deliveries dispatched previously. Cancelling a consumer will not discard them.
