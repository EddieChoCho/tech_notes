## RabbitMQ: Introducing Publisher Confirms

* Transactions(which can be unacceptably slow) v.s. Lightweight Publisher Confirms(an extension to AMQP)

## SpringAMQP - Publisher Confirms and Returns

* Set `publisherConfirmType` property of CachingConnectionFactory to `ConfirmType.CORRELATED`.
* Obtained a channel instances created by the factory are wrapped in an PublisherCallbackChannel, which is used to
  facilitate the callbacks.
* Register a PublisherCallbackChannel.Listener with the Channel. (The PublisherCallbackChannel implementation contains
  logic to route a "confirm" or return to the appropriate listener.)

### Publishing is Asynchronous - How to Detect Successes and Failures

* Publishing messages is an asynchronous mechanism and, by default, messages that cannot be routed are dropped by
  RabbitMQ.
* Consider two failure scenarios:
    * Publish to an exchange but there is no matching destination queue.
    * Publish to a non-existent exchange.

#### Publish to a non-existent exchange

* The message is dropped and no return is generated.
* The underlying channel is closed with an exception.
* Solution-1: Obtain notifications of such events by registering a ChannelListener
  * By default, this exception is logged, but you can register a ChannelListener with the CachingConnectionFactory to
    obtain notifications of such events.

  ```
  this.connectionFactory.addConnectionListener(new ConnectionListener() {

    @Override
    public void onCreate(Connection connection) {
    }

    @Override
    public void onShutDown(ShutdownSignalException signal) {
        ...
    }

  });
  ```

  * You can examine the signal’s reason property to determine the problem that occurred.
  * To detect the exception on the sending thread, you can setChannelTransacted(true) on the RabbitTemplate and the
    exception is detected on the txCommit().
  * However, `transactions significantly impede performance`, so consider this carefully before enabling transactions
    for just this one use case.

* Solution-2: publisher confirms
  * The RabbitTemplate implementation of AmqpTemplate supports publisher confirms and returns.
  * For publisher confirms (also known as publisher acknowledgements), the template requires a CachingConnectionFactory
    that has its publisherConfirm property set to ConfirmType.CORRELATED.
  * Confirms are sent to the client by it registering a RabbitTemplate.ConfirmCallback by calling setConfirmCallback(
    ConfirmCallback callback).
    * Only one ConfirmCallback is supported by a RabbitTemplate.
  * The callback must implement this
    method: ```void confirm(CorrelationData correlationData, boolean ack, String cause);```
    * correlationData: an object supplied by the client when sending the original message.
    * ack: true for an ack and false for a nack.
    * cause: for nack instances, the cause may contain a reason for the nack, if it is available when the nack is
      generated.
      * e.g., When sending a message to a non-existent exchange. In that case, the broker closes the channel. The reason
        for the closure is included in the cause.

### Publish to an exchange but there is no matching destination queue

* The RabbitTemplate implementation of AmqpTemplate supports publisher confirms and returns
* For returned messages, the template’s mandatory property must be set to true or the mandatory-expression must evaluate
  to true for a particular message.
* This feature requires a CachingConnectionFactory that has its publisherReturns property set to true (see Publisher
  Confirms and Returns).
* Returns are sent to the client by it registering a RabbitTemplate.ReturnsCallback by calling setReturnsCallback(
  ReturnsCallback callback).
  * Only one ReturnsCallback is supported by each RabbitTemplate.
* The callback must implement the following method: ```void returnedMessage(ReturnedMessage returned);```
* The ReturnedMessage has the following properties:
  * message - the returned message itself
  * replyCode - a code indicating the reason for the return
  * replyText - a textual reason for the return - e.g. NO_ROUTE
  * exchange - the exchange to which the message was sent
  * routingKey - the routing key that was used

## References

* [RabbitMQ - Introducing Publisher Confirms](https://blog.rabbitmq.com/posts/2011/02/introducing-publisher-confirms)
* [SpringAMQP - Publisher Confirms and Returns](https://docs.spring.io/spring-amqp/docs/current/reference/html/#cf-pub-conf-ret)