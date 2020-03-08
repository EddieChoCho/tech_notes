# Spring for Apache Kafka

## MessageListenerContainer

* KafkaMessageListenerContainer v.s. ConcurrentMessageListenerContainer
    * The KafkaMessageListenerContainer receives all message from all topics or partitions on a single thread. 
    * The ConcurrentMessageListenerContainer delegates to one or more KafkaMessageListenerContainer instances to provide multi-threaded consumption.
    
* RecordInterceptor (since version 2.2.7)
    * It will be invoked before calling the listener allowing inspection or modification of the record. 
    * If the interceptor returns null, the listener is not called. 
    * The interceptor is not invoked when the listener is a batch listener. (No interceptor is provided for batch listeners because Kafka already provides a ConsumerInterceptor.)
    * Starting with version 2.3, the CompositeRecordInterceptor can be used to invoke multiple interceptors.
    * By default, when using transactions, the interceptor is invoked after the transaction has started. Starting with version 2.3.4, you can set the listener container’s interceptBeforeTx property to invoke the interceptor before the transaction has started instead.

* The parameters of KafkaMessageListenerContainer 
    * ConsumerFactory
    * ContainerProperties(topicPartitionOffsets(TopicPartitionOffset)/topics(String)/topicPattern(Pattern))

* To assign a MessageListener to a container 
    * You can use the ContainerProps.setMessageListener method when creating the Container.

### ConcurrentMessageListenerContainer

* TBD...

### Committing Offsets

* Kafka auto-commits the offsets if `enable.auto.commit` consumer property is `true`.
    * Note: `auto.commit.interval.ms`,`spring.kafka.consumer.auto-commit-interval`
* If `enable.auto.commit` consumer property is `false`, the containers support several AckMode settings.
* The consumer poll() method returns one or more ConsumerRecords. The MessageListener is called for each record. 
* The following lists describes the action taken by the container for each AckMode (when transactions are not being used):
    * RECORD: Commit the offset when the listener returns after processing the record.
    * BATCH(default): Commit the offset when all the records returned by the poll() have been processed.
    * TIME: Commit the offset when all the records returned by the poll() have been processed, as long as the ackTime since the last commit has been exceeded.
    * COUNT: Commit the offset when all the records returned by the poll() have been processed, as long as ackCount records have been received since the last commit.
    * COUNT_TIME: Similar to TIME and COUNT, but the commit is performed if either condition is true.
    * MANUAL: The message listener is responsible to acknowledge() the Acknowledgment. After that, the same semantics as BATCH are applied.
        * Require the listener to be an `AcknowledgingMessageListener `or a `BatchAcknowledgingMessageListener`.
    * MANUAL_IMMEDIATE: Commit the offset immediately when the Acknowledgment.acknowledge() method is called by the listener.
        * Require the listener to be an `AcknowledgingMessageListener `or a `BatchAcknowledgingMessageListener`.
* When using transactions, the offset(s) are sent to the transaction and the semantics are equivalent to RECORD or BATCH, depending on the listener type (record or batch).

#### Acknowledgment

* acknowledge(): This method gives the listener control over when offsets are committed.
* nack(long sleep): This method is used with a record listener
    * when nack() is called, any pending offsets are committed, the remaining records from the last poll are discarded, and seeks are performed on their partitions so that the failed record and unprocessed records are redelivered on the next poll(). 
    * The consumer thread can be paused before redelivery, by setting the sleep argument. This is similar functionality to throwing an exception when the container is configured with a SeekToCurrentErrorHandler.
* nack(int index, long sleep): This method is used with a batch listener.
    * You can specify the index within the batch where the failure occurred. 
    * When nack() is called, offsets will be committed for records before the index and seeks are performed on the partitions for the failed and discarded records so that they will be redelivered on the next poll(). 
    * This is an improvement over the SeekToCurrentBatchErrorHandler, which can only seek the entire batch for redelivery.
* Note: * `nack` can only be called on the consumer thread that invokes your listener. 
* [Seek To Current Container Error Handlers](https://docs.spring.io/spring-kafka/reference/html/#seek-to-current)
* Note: When using partition assignment via group management, it is important to ensure the sleep argument (plus the time spent processing records from the previous poll) is less than the consumer max.poll.interval.ms property.

### @KafkaListener
* [Manually Assigning All Partitions.](https://docs.spring.io/spring-kafka/reference/html/#tip-assign-all-parts)

#### @KafkaListener Lifecycle Management


## References
* [Spring for Apache Kafka](https://docs.spring.io/spring-kafka/reference/html/#reference)

## Further Readings
* [Understanding the ‘enable.auto.commit’ Kafka Consumer property](https://medium.com/@danieljameskay/understanding-the-enable-auto-commit-kafka-consumer-property-12fa0ade7b65)
* [How Kafka’s Consumer Auto Commit Configuration Can Lead to Potential Duplication or Data Loss](https://blog.newrelic.com/engineering/kafka-consumer-config-auto-commit-data-loss/)


