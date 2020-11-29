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

## References
* [Reactive Architecture(1): Introduction to Reactive Systems](https://academy.lightbend.com/courses/course-v1:lightbend+LRA-IntroToReactive+v1/course/)









