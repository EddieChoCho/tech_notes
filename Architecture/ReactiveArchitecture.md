# Reactive Architecture
## 1. Introduction to Reactive Systems
### Why Reactive
* Why Reactive? The primary goal of reactive architecture is to provide an experience that is responsive under all conditions. 

* The software landscape problem that reactive architecture is addressing
	* The scale of modern systems far exceeds that of legacy systems, consisting of hundreds or even thousands of nodes.
* Data at rest
	* The data is not consumed at the time it is injested. It is stored, and then consumed later in a batch process.
* Streaming data?
	* A firehose of data. Data can flow through the system at any time with no upper bound on the amount of data.

* What we're trying to actually solve is a change in modern user expectations -- The software will be available when they need it. There is no downtime.

### The Reactive Principles
#### [Reactive Manifesto](http://www.reactivemanifesto.org/)
* Responsive
	* Our primary end goal.
	* A ractive system consistently responds in a timely fashion.
	* Responsiveness is the cornerstone of usability.
	* Systems must respond in a fast and consistent fashion if at all possible.
	* Responsive systems build user confidence.

* Resilient
	* A reactive system remains responsive, even when a failure occurs. Resilience provides responsiveness despite any failures. 
	* Achieved through replication, isolation, containment and delegation. 
		* Replication: have multiple copies
		* Isolation: services can function on their own
		* Containment: A consequence of isolation: it means that if there is a failure it doesn't propagate to another service because it is isolated.
		* Delegation: recovery is managed by an external component. 
	* Failures are isolated into a single component. 
	* Recovery is delegated to an external component. (If you are the failing component, then you're not reliable enough at least to handle your own failure. e.g. If the system goes down the system can't restart itself because it's down)

* Elastic
	* A reactive system remains responsive, despite changes to system load. Elasticity provides responsiveness despite increases or decreases in load.
	* Elasticity implies that we can not only scale up when needed but then when the load decreases we can scale back down in order to conserve resources.
	* Implies zero contention and no central bottlenecks. (This is the goal)
	* Predictive auto scaling techniques can be used to support elasticity.
	* Scaling up provide responsiveness during peak, while scaling down improves cost effectiveness.

* Message Driven
	* A reactive system is built on a foundation of asynchronous non-blocking messages. 
	* Not Event Driven, there are other types of messages which are not the events of a typical event-driven system.
	* Responsiveness, Resilience, Elasticity are all supported by a Message Driven Architecture. 
	* Messages are asynchronous and non-blocking.
	* Provides loose coupling, isolation, location transparency.
	* Resources are consumed only while active. 
		* You don't have a situation where you make a request and that request is blocking. And so you're stuck waiting for a response from somebody else who may take a long time or may not respond at all.
		* If you're stuck waiting and while you're waiting you're potentially consuming a thread, memory, CPU cycles, and/or any number of physical resources because you're stuck waiting.

### Reactive Systems vs Reactive Programming 

#### Reactive Programming
* Reactive Programming can be used to support the construction of Reactive Systems.
* Supports breaking problems into small, discrete steps.
* Steps are executed in an async/non-blocking fashion, usually via a callback mechanism.
	* e.g. Futures/Promises, Streams, RxJava/RxScala.
* A system that uses Reactive Programming is not necessarily a Reactive System.

#### The Actor Model
* The actor model is a programming paradigm that supports the construction of reactive systems.
* It is Message Driven
* Abstractions provide Elasticity and Resilience.
* It can be used to build Responsive software.
* On the JVM
	* Akka implements the Actor Model.
	* Akka is the foundation of Reactive Tools like Lagom and Akka Streams.

##### Fundamental Concepts of the Actor Model
* All Computation occurs inside of Actors.
* Each actor has an address.
* Actors communicate only through asynchronous messages.

##### Location Transparency
* The Message Driven nature of Actors supports Location Transparency.
* Actors communicate using the same technique, regardless of location.
* Local v.s. Remote is mostly configuration. (The API is identical no matter whether it's sending to a remote actor or a local actor. )
* Location Transparency enables actors to be both Resilient and Elastic.

##### Location Transparency v.s.Transparent Remoting
* Transparent Remoting
    * Remote calls like local calls. 
    * Hides the fact that you are making remote calls.
    * Hides potential failure scenarios (e.g. network failures).
* Location Transparency
    * Local calls like remote calls. 
    * Assumes you are always making remote calls.
    * You have to assume remote failure scenarios can occur (e.g. network failures).

##### Importance of the Actor Model
* There are many reactive programming tools. (e.g. Futures, Streams, RxJava, RxScala...)
* Most of them support only some of the Reactive principles.
* You often have to combine different technologies to build a Reactive System.
* The Actor Model provides facilities to support all of the Reactive principles.
    * Message Driven by default.
    * Location Transparency to support Elasticity and Resilience through distribution.
    * Elasticity and Resilience provide Responsiveness.

##### Reactive System Without Actors
* Reactive Systems are possible without using actors.
* Components are added on rather than being built in.
* Requires additional infrastructure such as:
    * Service Registry
    * Load Balancer
    * Message Bus
* Result will be Reactive at the large scale, not necessarily the small.


## References
* [Reactive Architecture(1): Introduction to Reactive Systems](https://academy.lightbend.com/courses/course-v1:lightbend+LRA-IntroToReactive+v1/course/)









