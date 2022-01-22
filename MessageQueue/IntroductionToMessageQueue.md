# Introduction to Message Queue

## Issues that we will face when dealing with cross services communication:

* Need to implement our re-try mechanism to handle the communication failure scenarios.
  * [Spring Retry](https://www.baeldung.com/spring-retry)
  * [Automatically Retry Failed Jobs in Quartz](https://github.com/EddieChoCho/tech_notes/blob/master/Java/Quartz.md#automatically-retry-failed-jobs-in-quartz2)

* Lack of Resilience: If one of the services is down, then users' operation will be blocked.
* Response time: User might need to wait for long RTT across multiple services.
* Coupling: Services need to know each other. Services need to explicitly know how to reach another service and how to
  handle the responses.

## How does the message queue cooperate with services?

* Publisher: Send messages to a broker
* Broker:
  * Receive messages from publishers, sending messages to consumers.
  * If a consumer is not available(e.g., crashed), messages will be queued in the broker. And be delivered to the
    consumer when it is recovered.
* Consumer: Receive messages from a broker.

## With message queue, we can solve the those issues by:

* Message queue asynchronous communication pattern: An application may publish information for any number of clients,
  and does not need to wait for responses.
* No need to implement for handling failure scenarios of receivers.
* Better resilience
* Faster response time
* Decoupling

## References

* [When Microservices Meet Event Sourcing](https://youtu.be/cISNDnwlSgw)
* [AWS - What is a Message Queue?](https://aws.amazon.com/message-queue/)

# Communication between broker and consumer

## Acknowledgment

* Auto acknowledgment
  * RabbitMQ can consider a message to be successfully delivered either immediately after it is sent out (written to a
    TCP socket)

* Manual acknowledgment
  * Manually sent acknowledgements can be positive or negative and use one of the following protocol methods:
    * basic.ack is used for positive acknowledgements
    * basic.nack is used for negative acknowledgements (note: this is a RabbitMQ extension to AMQP 0-9-1)
    * basic.reject is used for negative acknowledgements but has one limitation compared to basic.nack

* Auto v.s. Manual Better throughput v.s. Flexible receive and re-delivery mechanism

## The order of the messages(RabbitMQ)

* Default: FIFO(Queue)
* Ordering also can be affected by the presence of:
  * multiple competing consumers
  * consumer priorities
  * message redeliveries
* When a message is re-queued, it will be placed to its original position in its queue, if possible.
* If not (due to concurrent deliveries and acknowledgements from other consumers when multiple consumers share a queue),
  the message will be requeued to a position closer to queue head.

* Try to implement features which does not relay on the order of the message. -> Easy to scale-out.
* If we need to have the strict ordering messages -> more restrictions. e.g., a queue can only be consumed by one
  consumer, could not use batch consuming methods.
