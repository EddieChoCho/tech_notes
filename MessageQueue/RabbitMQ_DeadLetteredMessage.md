# Handle Dead-lettered Messages with RabbitMQ

## When is a message considered dead?
* A message considered dead where a message becomes undeliverable after reaching RabbitMQ.
* By default, the broker drops dead messages, then the messages become dead-lettered.
* There are four identified situations where a message becomes dead-lettered.
    * The message is negatively acknowledged by a consumer using basic.reject or basic.nack with requeue parameter set to false.
    * The message expires due to per-message TTL.
    * The message is dropped because its queue exceeded a length limit
    * For Quorum Queues, when a message has been returned more times than the limit(`x-delivery-limit` header) the message will be dropped.

## How to handle the dead-letter?  
* Queues attached to a `dead letter exchange`(DLX) collect dropped messages,
    * Attach a DLX when you know that you have messages that might be dropped, but still needs to be handled.    
    * DLXs are normal exchanges. They can be any of the usual types and are declared as usual.
    * DLX can be defined by clients using the queue's arguments, or in the server using policies.
    * DLX arguments/keys:
        * x-dead-letter-exchange/dead-letter-exchange: 
            * The name of the exchange that wanted to be attached with a queue/queues as a DLX.
        * x-dead-letter-routing-key/dead-letter-routing-key: 
            * You may also specify a routing key to be used when dead-lettering messages. 
            * If this is not set, the message's own routing keys will be used.

* To handle the dead-letter, we need to receive the messages from DLX.
    * We can bind queues(`dead letter queue` - DLQ) to a DLX with different routing keys.
    * It's up to you to decide how to handle messages in the dead letter queue. When implemented correctly, information is almost never lost.

## Dead-Lettered Effects on Messages
* Dead-lettering a message modifies its headers.
    * the exchange name is replaced with that of the latest dead-letter exchange,
    * the routing key may be replaced with that specified in a queue performing dead lettering,
    * ...TBD
 
* The dead-lettering process adds an array to the header of each dead-lettered message named x-death
    * This header is a table that consists of several fields, such as: queue, reason(rejected, expired, maxlen, or delivery-limit), exchange, routing-keys, count...etc.

* Three top-level headers are added for the very first dead-lettering event.
    * x-first-death-reason, x-first-death-queue, x-first-death-exchange

## Safety
* Dead-lettered messages are re-published without publisher confirms turned on internally. Therefore using DLX in a clustered RabbitMQ environment is not guaranteed to be safe. 
* Messages are removed from the original queue immediately after publishing to the DLX target queue. 
    * This ensures that there is no chance of excessive message buildup that could exhaust broker resources.
    * But messages can be lost if the target queue isn't available to accept messages. 
 

## References
* [RabbitMQ: Dead Letter Exchanges](https://www.rabbitmq.com/dlx.html)
* [RabbitMQ: Quorum Queues](https://www.rabbitmq.com/quorum-queues.html)
* [FAQ: When and how to use the RabbitMQ Dead Letter Exchange](https://www.cloudamqp.com/blog/when-and-how-to-use-the-rabbitmq-dead-letter-exchange.html)