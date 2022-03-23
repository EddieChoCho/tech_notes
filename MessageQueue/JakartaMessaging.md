# Jakarta Messaging (formerly Java Message Service)

## 45 Java Message Service Concepts

### 45.1 Overview of the JMS API

#### 45.1.1 What Is Messaging?

* Messaging is a method of communication between software components or applications.
* Messaging enables distributed communication that is loosely coupled. A component sends a message to a destination, and
  the recipient can retrieve the message from the destination.
    * What makes the communication loosely coupled is that the destination is all that the sender and receiver have in
      common.
        * The sender and the receiver do not have to be available at the same time in order to communicate.
        * The sender does not need to know anything about the receiver; nor does the receiver need to know anything
          about the sender.
        * The sender and the receiver need to know only which message format and which destination to use.

#### 45.1.2 What Is the JMS API? the Java API that allows applications to create, send, receive, and read messages.

* JMS enables communication that is not only loosely coupled but also
    * Asynchronous:
        * A receiving client does not have to receive messages at the same time the sending client sends them.
        * The sending client can send them and go on to other tasks; the receiving client can receive them much later.
    * Reliable:
        * A messaging provider that implements the JMS API can ensure that a message is delivered once and only once.
        * Lower levels of reliability are available for applications that can afford to miss messages or to receive
          duplicate messages.

#### 45.1.3 When Can You Use the JMS API?

* An enterprise application provider is likely to choose a messaging API over a tightly coupled API, such as a remote
  procedure call (RPC), under the following circumstances.
    * The provider wants the components not to depend on information about other components' interfaces, so components
      can be easily replaced.
    * The provider wants the application to run whether all components are up and running simultaneously or not.
    * The application business model allows a component to send information to another and to continue to operate
      without receiving an immediate response.

#### 45.1.4 How Does the JMS API Work with the Java EE Platform?

* The JMS API in the Java EE platform has the following features.
    * Application clients, Enterprise JavaBeans (EJB) components, and web components can send or synchronously receive a
      JMS message. Application clients can in addition set a message listener that allows JMS messages to be delivered
      to it asynchronously by being notified when a message is available.
    * Message-driven beans, which are a kind of enterprise bean, enable the asynchronous consumption of messages in the
      EJB container. An application server typically pools message-driven beans to implement concurrent processing of
      messages.
    * Message send and receive operations can participate in Java Transaction API (JTA) transactions, which allow JMS
      operations and database accesses to take place within a single transaction.

### 45.2 Basic JMS API Concepts

#### 45.2.1 JMS API Architecture

* `JMS provider`
    * A JMS provider is a messaging system that implements the JMS interfaces and provides administrative and control
      features.
    * An implementation of the Java EE platform that supports the full profile includes a JMS provider.
* `JMS client`
    * JMS clients are the programs or components, written in the Java programming language, that produce and consume
      messages. Any Java EE application component can act as a JMS client.
* `Message`
    * Messages are the objects that communicate information between JMS clients.
* `Administered object`
    * Administered objects are JMS objects configured for the use of clients. The two kinds of JMS administered objects
      are destinations and connection factories, described in JMS Administered Objects.
    * An administrator can create objects that are available to all applications that use a particular installation of
      GlassFish Server; alternatively, a developer can use annotations to create objects that are specific to a
      particular application.

#### 45.2.2 Messaging Styles

* Point-to-Point Messaging Style
    * A point-to-point (PTP) product or application is built on the concept of `message queues`, senders, and receivers.
    * Each message is addressed to a specific queue, and receiving clients extract messages from the queues established
      to hold their messages.
    * PTP messaging has the following characteristics:
        * Each message has only one consumer.
        * The receiver can fetch the message whether it was running when the client sent the message or not.
    * `Use PTP messaging when every message you send must be processed successfully by one consumer`.

* Publish/Subscribe Messaging Style
    * In a publish/subscribe (pub/sub) product or application, `clients address messages to a topic`, which functions
      somewhat like a bulletin board.
    * Topics retain messages only as long as it takes to distribute them to subscribers.
    * Consumer v.s. Subscription
        * The consumer is a JMS object within an application, while the subscription is an entity within the JMS
          provider.
        * Normally, a topic can have many consumers, but a subscription has only one subscriber.
            * It is possible, however, to create shared subscriptions.
    * Pub/sub messaging has the following characteristics:
        * Each message can have multiple consumers.
        * A client that subscribes to a topic can consume only messages sent after the client has created a
          subscription, and `the consumer must continue to be active in order for it to consume messages`.

    * The JMS API allowing applications to create `durable subscriptions`, which receive messages sent while the
      consumers are not active.
        * Durable subscriptions provide the flexibility and reliability of queues but still allow clients to send
          messages to many recipients.
    * `Use pub/sub messaging when each message can be processed by any number of consumers (or none)`.

#### 45.2.3 Message Consumption

* Messages can be consumed in either of two ways:
    * Synchronously
        * A consumer explicitly fetches the message from the destination by calling the `receive` method.
        * The `receive` method can block until a message arrives or can time out if a message does not arrive within a
          specified time limit.

    * Asynchronously
        * An application client or a Java SE client can register a message listener with a consumer. A message listener
          is similar to an event listener.
        * Whenever a message arrives at the destination, the JMS provider delivers the message by calling the listener's
          onMessage method, which acts on the contents of the message.
        * In a Java EE application, a message-driven bean serves as a message listener (it too has an onMessage method),
          but a client does not need to register it with a consumer.

### 45.3 The JMS API Programming Model

#### 45.3.1 JMS Administered Objects

* Two parts of a JMS application, destinations and connection factories, are commonly maintained administratively rather
  than programmatically.
* JMS clients access administered objects through interfaces that are portable, so a client application can run with
  little or no change on more than one implementation of the JMS API.
* Ordinarily, an administrator configures administered objects in a JNDI namespace, and JMS clients then access them by
  using resource injection.

* JMS Connection Factories
    * A connection factory is the object a client uses to create a connection to a provider which encapsulates a set of
      connection configuration parameters that has been defined by an administrator.
    * Each connection factory is an instance of the ConnectionFactory, QueueConnectionFactory, or TopicConnectionFactory
      interface.
    * A Java EE server must provide a JMS connection factory with the logical JNDI name java:
      comp/DefaultJMSConnectionFactory. The actual JNDI name will be implementation-specific.
    * The following code fragment looks up the default JMS connection factory and assigns it to a ConnectionFactory
      object:
      ```
          @Resource(lookup = "java:comp/DefaultJMSConnectionFactory")
          private static ConnectionFactory connectionFactory;
      ```

* JMS Destinations
    * A destination is the object a client uses to specify the target of messages it produces and the source of messages
      it consumes.
    * In the PTP messaging style, destinations are called queues.
    * In the pub/sub messaging style, destinations are called topics.
    * A JMS application can use multiple queues or topics (or both).
    * In addition to injecting a connection factory resource into a client program, you usually inject a destination
      resource.
    * The following code specifies two resources, a queue and a topic. The resource names are mapped to destination
      resources created in the JNDI namespace:
      ```
          //JMS administered objects are normally placed in the jms naming subcontext.
          @Resource(lookup = "jms/MyQueue")
          private static Queue queue;
  
          @Resource(lookup = "jms/MyTopic")
          private static Topic topic;
      ```
    * The behavior of the application will depend on the kind of destination you use and not on the kind of connection
      factory you use.
        * e.g., You can inject a QueueConnectionFactory resource and use it with a Topic, and you can inject a
          TopicConnectionFactory resource and use it with a Queue.

#### 45.3.2 Connections

* A connection encapsulates a virtual connection with a JMS provider. For example, a connection could represent an open
  TCP/IP socket between a client and a provider service daemon.
* You use a connection to create one or more sessions.
    * The ability to create multiple sessions from a single connection is limited to application clients.
    * In web and enterprise bean components, a connection can create no more than one session.
    * You normally create a connection by creating a JMSContext object.

#### 45.3.3 Sessions

* A session is a single-threaded context for producing and consuming messages.
* You normally create a session (as well as a connection) by creating a JMSContext object.
* You use sessions to create message producers, message consumers, messages, queue browsers, and temporary destinations.
* Sessions serialize the execution of message listeners.
* A session provides a transactional context with which to group a set of sends and receives into an atomic unit of
  work.

#### 45.3.4 JMSContext Objects

* A JMSContext object combines a connection and a session in a single object.
    * That is, it provides both an active connection to a JMS provider and a single-threaded context for sending and
      receiving messages.
* You use the JMSContext to create the following objects: Message producers, Message consumers, Messages, Queue
  browsers, Temporary queues and topic
* `You can create a JMSContext in a try-with-resources block`.
    * Make sure that your application completes all its JMS activity within the try-with-resources block. Or you can
      call the close method on the JMSContext to close the connection.
    * You must close the connection when the application has finished its work.
* To create a JMSContext, call the createContext method on the connection factory:
    ```
      JMSContext context = connectionFactory.createContext();
      // When there is no active JTA transaction in progress, the createContext method creates a non-transacted session with an acknowledgment mode of JMSContext.AUTO_ACKNOWLEDGE. 
      // When there is an active JTA transaction in progress, the createContext method creates a transacted session. 
    ```
* You can also call the createContext method with the argument JMSContext.SESSION_TRANSACTED to create a transacted
  session:
    ```
      JMSContext context = connectionFactory.createContext(JMSContext.SESSION_TRANSACTED);
      //The session uses local transactions
    ```
* When you use a JMSContext, message delivery normally begins as soon as you create a consumer.

#### 45.3.5 JMS Message Producers

* A message producer is an object that is created by a JMSContext or a session and used for sending messages to a
  destination.
* You could create it this way:
    ```
      try (JMSContext context = connectionFactory.createContext();) {
      JMSProducer producer = context.createProducer();
      ...
    ```
* A JMSProducer is a lightweight object that does not consume significant resources.
    * For this reason, you do not need to save the JMSProducer in a variable; you can create a new one each time you
      send a message.
* You send messages to a specific destination by using the send method:
    ```
      context.createProducer().send(dest, message);
      // You can create the message in a variable before sending it, as shown here, or you can create it within the send call.
    ```

#### 45.3.6 JMS Message Consumers

* A message consumer is an object that is created by a JMSContext or a session and used for receiving messages sent to a
  destination.
* You could create it this way:
    ```
      try (JMSContext context = connectionFactory.createContext();) {
          JMSConsumer consumer = context.createConsumer(dest);
          ...
    ```
* A message consumer allows a JMS client to register interest in a destination with a JMS provider. The JMS provider
  manages the delivery of messages from a destination to the registered consumers of the destination.
* When you use a JMSContext to create a message consumer, message delivery begins as soon as you have created the
  consumer.
    * You can disable this behavior by calling setAutoStart(false) when you create the JMSContext and then calling the
      start method explicitly to start message delivery.
* If you want to stop message delivery temporarily without closing the connection, you can call the stop method; to
  restart message delivery, call start.
* You use the `receive` method to consume a message synchronously.
    ```
      // If you specify no arguments or an argument of 0, the method blocks indefinitely until a message arrives:
      Message m = consumer.receive();
      Message m = consumer.receive(0);
      // Use a synchronous receive with a timeout: Call the receive method with a timeout argument greater than 0. One second is a recommended timeout value:
      Message m = consumer.receive(1000); 
    ```
* You can use the JMSContext.createDurableConsumer method to create a durable topic subscription. This method is valid
  only if you are using a topic.

##### 45.3.6.1 JMS Message Listeners

* A message listener is an object that acts as an asynchronous event handler for messages.
* This object implements the MessageListener interface, which contains one method, onMessage.
* In the onMessage method, you define the actions to be taken when a message arrives.
* You register the message listener with a specific message consumer by using the setMessageListener method.
    ```
        consumer.setMessageListener(myListener);
    ```
* When message delivery begins, the JMS provider automatically calls the message listener's onMessage method whenever a
  message is delivered.
* In the Java EE web or EJB container, you use message-driven beans for asynchronous message delivery. A message-driven
  bean also implements the MessageListener interface.
* Your onMessage method should handle all exceptions. Throwing a RuntimeException is considered a programming error.

##### 45.3.6.2 JMS Message Selectors

* If your messaging application needs to filter the messages it receives, you can use a JMS message selector.
* It allows a message consumer for a destination to specify the messages that interest it.
* Message selectors assign the work of filtering messages to the JMS provider rather than to the application.
* A message selector is a String that contains an expression. The syntax of the expression is based on a subset of the
  SQL92 conditional expression syntax.
    ```
      // selects any message that has a NewsType property that is set to the value 'Sports' or 'Opinion':
      NewsType = 'Sports' OR NewsType = 'Opinion'
    ```
* A message selector cannot select messages on the basis of the content of the message body.
* The methods for creating shared consumers(e.g., createConsumer and createDurableConsumer), allow you to specify a
  message selector as an argument when you create a message consumer.
    * The message consumer then receives only messages whose headers and properties match the selector.

##### 45.3.6.3 Consuming Messages from Topics

* The semantics of consuming messages from topics are more complex than the semantics of consuming messages from queues.
* An application consumes messages from a topic by creating a subscription on that topic and creating a consumer on that
  subscription.
    * Subscriptions may be durable or non-durable, and they may be shared or unshared.
* A subscription will receive a copy of every message that is sent to the topic after the subscription is created,
  unless a message selector is specified.

* Unshared subscription
    * Unshared subscriptions are restricted to a single consumer. In this case, all the messages in the subscription are
      delivered to that consumer.

* Shared subscription
    * Shared subscriptions allow multiple consumers. In this case, each message in the subscription is delivered to only
      one consumer.

* Non-durable subscription
    * A non-durable subscription exists only as long as there is an active consumer on the subscription.
    * This means that any messages sent to the topic will be added to the subscription only while a consumer exists and
      is not closed.
    * A non-durable subscription may be either unshared or shared:
        * Unshared non-durable subscription
            * An unshared non-durable subscription does not have a name and may have only a single consumer object
              associated with it.
            * It is created automatically when the consumer object is created.
            * It is not persisted and is deleted automatically when the consumer object is closed.
            * The ```JMSContext.createConsumer``` method creates a consumer on an unshared non-durable subscription if a
              topic is specified as the destination.
        * Shared non-durable subscription
            * A shared non-durable subscription is identified by name and an optional client identifier, and may have
              several consumer objects consuming messages from it.
            * It is created automatically when the first consumer object is created.
            * It is not persisted and is deleted automatically when the last consumer object is closed.

* Durable subscription
    * A durable subscription is persisted and continues to accumulate messages until explicitly deleted, even if there
      are no consumer objects consuming messages from it.

##### 45.3.6.4 Creating Durable Subscriptions

* A durable subscription may be either unshared or shared:
    * Unshared durable subscription
        * An unshared durable subscription is identified by name and client identifier (which must be set) and may have
          only a single consumer object associated with it.
        * An unshared durable subscription can have only one active consumer at a time.
    * Shared durable subscription
        * A shared durable subscription is identified by name and an optional client identifier, and may have several
          consumer objects consuming messages from it.
* Inactive subscription
    * A durable subscription that exists but that does not currently have a non-closed consumer object associated with
      it is described as being inactive.

* A consumer identifies the durable subscription from which it consumes messages by specifying a unique identity that is
  retained by the JMS provider.
* Subsequent consumer objects that have the same identity resume the subscription in the state in which it was left by
  the preceding consumer.
* If a durable subscription has no active consumer, the JMS provider retains the subscription's messages until they are
  received by the subscription or until they expire.
* You establish the unique identity of an unshared durable subscription by setting the following:
    * A client ID for the connection
        * You can set the client ID administratively for a client-specific connection factory using either the command
          line or the Administration Console.
        * In an application client or a Java SE client, you can instead call JMSContext.setClientID.
    * A topic and a subscription name for the subscription

* The subscription becomes active after you create the consumer. for example:
    ```
    String subName = "MySub"; // A string that specifies the name of the subscription
    JMSConsumer consumer = context.createDurableConsumer(myTopic, subName);
    ```
* The JMS provider stores the messages sent to the topic, as it would store messages sent to a queue.
* If the program or another application calls createDurableConsumer using the same connection factory and its client ID,
  the same topic, and the same subscription name:
    * Then the subscription is reactivated and the JMS provider delivers the messages that were sent while the
      subscription was inactive.
* To delete a durable subscription:
    ```
        // First close the consumer
        consumer.close();
        // Then call the unsubscribe method with the subscription name as the argument
        context.unsubscribe(subName);  
        // The unsubscribe method deletes the state the provider maintains for the subscription.
    ```
* If you use a shared durable subscription, the connection factory you use does not need to have a client identifier to
  create the subscription.
    ```
        JMSConsumer consumer = context.createSharedDurableConsumer(topic, "MakeItLast");
    ```

##### 45.3.6.5 Creating Shared Subscriptions

* With a shared subscription, messages will be distributed among multiple clients that use the same topic and
  subscription name.
* Each message sent to the topic will be added to every subscription (subject to any message selectors).
* But each message added to a subscription will be delivered to only one of the consumers on that subscription, so it
  will be received by only one of the clients.
* This feature can improve the scalability of Java EE application client applications and Java SE applications. (
  Message-driven beans share the work of processing messages from a topic among multiple threads.)

#### 45.3.7 JMS Messages

* A JMS message can have three parts: a header, properties, and a body. Only the header is required.

##### 45.3.7.1 Message Headers

* JMSMessageID: Value that uniquely identifies each message sent by a provider. (Set By JMS provider send method)
* JMSDestination: Destination to which the message is being sent(queue or topic). (Set By JMS provider send method)

##### 45.3.7.2 Message Properties

* You can create and set properties for messages if you need values in addition to those provided by the header fields.
* You can use properties to provide compatibility with other messaging systems, or you can use them to create message
  selectors.
* The JMS API provides some predefined property names that begin with `JMSX`.
* A JMS provider is required to implement only one of these, JMSXDeliveryCount (which specifies the number of times a
  message has been delivered); the rest are optional.

##### 45.3.7.3 Message Bodies

* The JMS API defines six different types of messages: TextMessage, MapMessage, BytesMessage, StreamMessage,
  ObjectMessage, Message.
* Each message type corresponds to a different message body. These message types allow you to send and receive data in
  different forms.
* The JMS API provides methods for creating messages of each type and for filling in their contents.
* For example, to create and send a TextMessage:
    ```
        TextMessage message = context.createTextMessage();
        message.setText("msg text");
        context.createProducer().send(message);
    ```
* At the consuming end, a message arrives as a generic Message object.
* You can then cast the object to the appropriate message type and use more specific methods to access the body and
  extract the message contents (and its headers and properties if needed).
* Instead of casting the message to a message type, you can call the getBody method on the Message, specifying the type
  of the message as an argument:
    ```
        Message m = consumer.receive()
        if (m instanceof TextMessage) {
            String message = m.getBody(String.class);
            System.out.println("Reading message: " + message);
        } else {
            // Handle error or process another message type
        }
    ```
* The JMS API provides shortcuts for creating and receiving a TextMessage, BytesMessage, MapMessage, or ObjectMessage.
* You can use the receiveBody method to receive any type of message `except StreamMessage and Message`.
    ```
        context.createProducer().send(dest, "This is a message");
        String message = receiver.receiveBody(String.class);
    ```

#### 45.3.8 JMS Queue Browsers

* The JMS API provides a QueueBrowser object that allows you to browse the messages in the queue and display the header
  values for each message.
* To create a QueueBrowser object, use the JMSContext.createBrowser method:
    ```
        QueueBrowser browser = context.createBrowser(queue);
    ```
* The createBrowser method allows you to specify a message selector as a second argument when you create a QueueBrowser.
* The JMS API provides no mechanism for browsing a topic.
* Messages usually disappear from a topic as soon as they appear.
    * If there are no message consumers to consume them, the JMS provider removes them.
    * Although durable subscriptions allow messages to remain on a topic while the message consumer is not active, JMS
      does not define any facility for examining them.

#### 45.3.9 JMS Exception Handling

* The root class for all checked exceptions in the JMS API is JMSException.
* The root cause for all unchecked exceptions in the JMS API is JMSRuntimeException.
* Catching JMSException and JMSRuntimeException provides a generic way of handling all exceptions related to the JMS
  API.

### 45.4 Using Advanced JMS Features

* Feature: every message be received once and only once.
* The most reliable way to produce a message is to send a PERSISTENT message, and to do so within a transaction.
* The most reliable way to consume a message is to do so within a transaction, either from a queue or from a durable
  subscription to a topic.
* PERSISTENT message
    * JMS messages are PERSISTENT by default; PERSISTENT messages will not be lost in the event of JMS provider failure.
* Transactions
    * A transaction is a unit of work into which you can group a series of operations, such as message sends and
      receives, so that the operations either all succeed or all fail.
* Some features primarily allow an application to improve performance. e.g.,
    * Message TTL, so that consumers do not receive unnecessary outdated information.
    * You can send messages asynchronously.

#### 45.4.1 Controlling Message Acknowledgment

* Until a JMS message has been acknowledged, it is not considered to be successfully consumed.
* The successful consumption of a message ordinarily takes place in three stages:
    * The client receives the message.
    * The client processes the message.
    * The message is acknowledged. Acknowledgment is initiated either by the JMS provider or by the client, depending on
      the session acknowledgment mode.

* In locally transacted sessions:
    * A message is acknowledged when the session is committed.
    * If a transaction is rolled back, all consumed messages are redelivered.

* In a JTA transaction:
    * A message is acknowledged when the transaction is committed.

* In non-transacted sessions, when and how a message is acknowledged depend on a value that may be specified as an
  argument of the createContext method:
    * JMSContext.AUTO_ACKNOWLEDGE
        * This setting is the default for application clients and Java SE clients.
        * The JMSContext automatically acknowledges a client's receipt of a message when:
            * The client has successfully returned from a call to receive.
            * Or the MessageListener it has called to process the message returns successfully.

        * A synchronous receive in a JMSContext that is configured to use auto-acknowledgment.
            * It is the one exception to the rule that message consumption is a three-stage process as described
              earlier.
            * In this case, the receipt and acknowledgment take place in one step, followed by the processing of the
              message.

    * JMSContext.CLIENT_ACKNOWLEDGE
        * A client acknowledges a message by calling the message's acknowledge method.
        * In this mode, acknowledgment takes place on the session level:
            * Acknowledging a consumed message automatically acknowledges the receipt of all messages that have been
              consumed by its session.
            * e.g., If a message consumer consumes ten messages and then acknowledges the fifth message delivered, all
              ten messages are acknowledged.

        * Note: In the Java EE platform, the JMSContext.CLIENT_ACKNOWLEDGE setting can be used only in an application
          client, not in a web component or enterprise bean.

    * JMSContext.DUPS_OK_ACKNOWLEDGE
        * This option instructs the JMSContext to lazily acknowledge the delivery of messages.
        * This is likely to `result in the delivery of some duplicate messages if the JMS provider fails`, so it should
          be used only by consumers that can `tolerate duplicate messages`.
            * If the JMS provider redelivers a message, it must set the value of the JMSRedelivered message header to
              true.
        * This option can reduce session overhead by minimizing the work the session does to prevent duplicates.

#### 45.4.2 Specifying Options for Sending Messages

* You can set a number of options when you send a message. These options enable you to perform the following tasks:
    * Specify that messages are persistent: meaning they must not be lost in the event of a provider failure.
    * Set priority levels for messages: which can affect the order in which the messages are delivered.
    * Specify an expiration time for messages: so they will not be delivered if they are obsolete.
    * Specify a delivery delay for messages: so that they will not be delivered until a specified amount of time has
      expired.

##### 45.4.2.1 Specifying Message Persistence

* The JMS API supports two delivery modes specifying whether messages are lost if the JMS provider fails:
    * PERSISTENT
        * Default mode,It instructs the JMS provider to take extra care to ensure that a message is not lost in transit
          in case of a JMS provider failure.
        * A message sent with this delivery mode is logged to stable storage when it is sent.

    * NON_PERSISTENT

##### 45.4.2.2 Setting Message Priority Levels

* The ten levels of priority range from 0 (lowest) to 9 (highest).
* If you do not specify a priority level, the default level is 4.
* A JMS provider tries to deliver higher-priority messages before lower-priority ones, but does not have to deliver
  messages in exact order of priority.

##### 45.4.2.3 Allowing Messages to Expire

* By default, a message never expires.
* If a message can become obsolete after a certain period(e.g., a message that contains rapidly changing data such as a
  stock price), however, you may want to set an expiration time.
* Use the setTimeToLive method of the JMSProducer interface to set a default expiration time for all messages sent by
  that producer.
    ```
        context.createProducer().setTimeToLive(300000).send(dest, msg);
    ```
* Any message not delivered before the specified expiration time is destroyed.
* The destruction of obsolete messages conserves storage and computing resources.

##### 45.4.2.4 Specifying a Delivery Delay

* You can use method chaining to set the delivery delay when you create a producer and send a message.
    ```
        context.createProducer().setDeliveryDelay(3000).send(dest, msg);
    ```

#### 45.4.5 Sending Messages Asynchronously

* Sending a message asynchronously involves supplying a callback object. You specify a CompletionListener with an
  onCompletion method.
* The CompletionListener class must implement two methods, onCompletion and onException. The onCompletion method is
  called if the `send` succeeds, and the onException method is called if it fails.

### 45.5 Using the JMS API in Java EE Applications

* Application components in the web and EJB containers must not attempt to create more than one active (not closed)
  Session object per connection.
* Multiple JMSContext objects are permitted, however, since they combine a single connection and a single session.
* This rule does not apply to application clients. The application client container supports the creation of multiple
  sessions for each connection.

## References

* [Oracle's Java EE 7 JMS tutorial](https://docs.oracle.com/javaee/7/tutorial/partmessaging.htm)