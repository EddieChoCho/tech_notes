## Consumer Acknowledgements and Publisher Confirms [1]
## Precondition of writing this note: I haven't used RabbitMQ before. I had tried to use Kafka as MQ. 


## References
* [1][RabbitMQ - Consumer Acknowledgements and Publisher Confirms](https://www.rabbitmq.com/confirms.html)

## Further Readings
* [Spring + Kafka + Acknowledgment to manually ack](https://docs.spring.io/spring-kafka/api/org/springframework/kafka/annotation/KafkaListener.html)
* [Kafka with selective acknowledgments (kmq) performance & latency benchmark](https://softwaremill.com/kafka-with-selective-acknowledgments-performance/)
* [Using Kafka as a message queue](https://softwaremill.com/using-kafka-as-a-message-queue/)
    * Apache Kafka is a very popular publish/subscribe system, which can be used to reliably process a stream of data. 
    * The central concept in Kafka is a topic, which can be replicated across a cluster providing safe data storage.
        * By committing processed message offsets back to Kafka, it is relatively straightforward to implement guaranteed “at-least-once” processing.
        * One important limitation: you can only commit - or, in other words, acknowledge processing of - all messages up to a given offset. It is not possible to acknowledge individual messages.
