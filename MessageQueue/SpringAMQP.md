# SpringAMQP

## Connection and Resource Management
* The central component for managing a connection to the RabbitMQ broker is the ConnectionFactory interface.

### Choosing a Connection Factory

* PooledChannelConnectionFactory
	* For most use cases, the PooledChannelConnectionFactory should be used. 
	* This factory manages a single connection and two pools of channels, based on the Apache Pool2. 
	* One pool is for transactional channels, the other is for non-transactional channels. The pools are GenericObjectPool s with default configuration; a callback is provided to configure the pools; 

* ThreadChannelConnectionFactory
	* The ThreadChannelConnectionFactory can be used if you want to ensure strict message ordering without the need to use Scoped Operations.
	* This factory manages a single connection and two ThreadLocal s, one for transactional channels, the other for non-transactional channels. 
	* This factory ensures that all operations on the same thread use the same channel (as long as it remains open). 
	* This facilitates strict message ordering without the need for Scoped Operations. To avoid memory leaks, if your application uses many short-lived threads, you must call the factory’s closeThreadChannel() to release the channel resource.

* CachingConnectionFactory
	* The CachingConnectionFactory should be used if you want to use correlated publisher confirmations or if you wish to open multiple connections, via its CacheMode.
 
### RabbitMQ Automatic Connection/Topology recovery
* Spring AMQP now uses the 4.0.x version of amqp-client, which has auto recovery enabled by default. The framework is completely compatible with auto-recovery being enabled.
* Spring AMQP can still use its own recovery mechanisms if you wish, disabling it in the client, (by setting the automaticRecoveryEnabled property on the underlying RabbitMQ connectionFactory to false). 
* Note: Only elements (queues, exchanges, bindings) that are defined as beans will be re-declared after a connection failure. Elements declared by invoking RabbitAdmin.declare*() methods directly from user code are unknown to the framework and therefore cannot be recovered.

## AmqpTemplate

### Publishing is Asynchronous — How to Detect Successes and Failures

* Publish to an exchange but there is no matching destination queue.
	* This is covered by publisher returns, as described in Correlated Publisher Confirms and Returns.
	
* Publish to a non-existent exchange.
	* The message is dropped and no return is generated.
	* The underlying channel is closed with an exception. By default, this exception is logged, but you can register a ChannelListener with the CachingConnectionFactory to obtain notifications of such events. 


## How to Consume Messages?
* Through AMQPTemplate
* Through @RabbitListener annotation
* Through MessageListener

 ### Configurations
* PrefetchCount 
* ConcurrentConsumers

## References
* [Spring AMQP](https://docs.spring.io/spring-amqp/docs/current/reference/html)
 
    