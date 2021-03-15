## Introduction to Akka
* Akka is a toolkit and runtime for building highly concurrent, distributed, and fault-tolerant event-driven applications on the JVM.

### Akka provides
* Multi-threaded behavior without the use of low-level concurrency constructs like atomics or locks — relieving you from even thinking about memory visibility issues.
* Transparent remote communication between systems and their components — relieving you from writing and maintaining difficult networking code.
* A clustered, high-availability architecture that is elastic, scales in or out, on demand — enabling you to deliver a truly reactive system.

### Actor
* Actors are the unit of execution in Akka. 
* The Actor model is an abstraction that makes it easier to write correct concurrent, parallel and distributed systems.

#### Benefits of using the Actor Model
* Event-driven model
	* Actors perform work in response to messages. Communication between Actors is asynchronous, allowing Actors to send messages and continue their own work without blocking to wait for a reply.
	* Question: What if an Actor crashed after it consume the message? Will the message be consumed by anthor Actor? What is the mechanism?
* Strong isolation principles
	* Unlike regular objects in Java, an Actor does not have a public API in terms of methods that you can invoke. Instead, its public API is defined through messages that the actor handles. 
	* This prevents any sharing of state between Actors; the only way to observe another actor’s state is by sending it a message asking for it.
* Location transparency
	* The system constructs Actors from a factory and returns references to the instances. 
	* Because location doesn’t matter, Actor instances can start, stop, move, and restart to scale up and down as well as recover from unexpected failures.
* Lightweight
	* Each instance consumes only a few hundred bytes, which realistically allows millions of concurrent Actors to exist in a single application.

### Defining Actors and messages
* Each actor defines a type T for the messages it can receive. 
* Case classes and case objects make excellent messages since they are immutable and have support for pattern matching, something we will take advantage of in the Actor when matching on the messages it has received.

* When defining Actors and their messages, keep these recommendations in mind:
	* Since messages are the Actor’s public API, it is a good practice to define messages with good names and rich semantic and domain specific meaning, even if they just wrap your data type. This will make it easier to use, understand and debug actor-based systems.
	* Messages should be immutable, since they are shared between different threads.
	* It is a good practice to put an actor’s associated messages as static classes in the AbstractBehaavior’s class. This makes it easier to understand what type of messages the actor expects and handles.
	* It is a good practice obtain an actor’s initial behavior via a static factory method.
	
## References
* [Introduction to Akka](https://doc.akka.io/docs/akka/2.6/typed/guide/introduction.html?language=java) 