# Designing Data-Intensive Applications

## Ch1. Reliable, Scalable, and Maintainable Applications
* Many applications need to:
	* Store data so that they, or another application, can find it again later (databases)
	* Remember the result of an expensive operation, to speed up reads (caches)
	* Allow users to search data by keyword or filter it in various ways (search indexes)
	* Send a message to another process, to be handled asynchronously (stream pro‐cessing)
	* Periodically crunch a large amount of accumulated data (batch processing)

* In this book, we focus on three concerns that are important in most software systems: Reliability, Scalability, and Maintainability

### Reliability
* The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error). 

### Scalability
* As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.
 
### Maintainability
* Over time, many different people will work on the system (engineering and operations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively.
