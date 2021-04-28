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
