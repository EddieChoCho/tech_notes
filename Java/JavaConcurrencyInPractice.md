# Java Concurrency In Practice 
## The introduction 
### Chapter 1:
#### Benefits of Threads
* Exploiting Multiple Processors
* Simplicity of Modeling
* Simplified Handling of Asynchronous Events
* More Responsive User Interfaces

#### Safety Hazards
#### Liveness Hazards
* Chapter 10 describes various forms of liveness failures and how to avoid them, 
	* deadlock (Section 10.1), 
	* starvation (Section 10.3.1), 
	* livelock (Section 10.3.3). 
#### Performance Hazards
## Part I - Fundamentals: 
### Chapter 2. Thread Safety
#### 2.1 What is thread-safe?
* A class is thread-safe if it behaves correctly when accessed from multiple threads, regardless of the scheduling or interleaving of the execution of those threads by the runtime ebvironment, and with no additional synchronization or other coordination on the part of the calling code.
* Thread-safe classes encapsulate any needed synchronization so that clients need not provide their own.
* Stateless objects are always thread-safe.
#### 2.2 Atomicity
    * Atomic: execute as a single, indivisible operation
* Race Conditions
    * A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime
    * Check-then-act race condition
        * You observe something to be true (file X doesn't exist) and then take action based on that observation (create X)
        * But in fact the observation could have become invalid between the time you observed it and the time you acted on it (someone else created X in the meantime), causing a problem (unexpected exception, overwritten data, file corruption).
    * Read-modify-write race condition
        * Like incrementing a counter, define a transformation of an object's state in terms of its previous state.    
    * Race condition v.s. Data race
        * Data race: synchronization is not used to coordinate all access to a shared non-final field.
        * Not all race conditions are data races, and not all data races are race conditions, but they both can cause concurrent programs to fail in unpredictable ways.
        
        
         
### Chapter 3. (Sharing Objects)
* TBD...
### Chapter 4. (Composing Objects)
* TBD...
### Chapter 5 (Building Blocks)
* TBD...
## Part II - Structuring Concurrent Applications:
### Chapter 6 (Task Execution)
* TBD...
### Chapter 7 (Cancellation and Shutdown)
* TBD... 
### Chapter 8 (Applying Thread Pools)
* TBD...
### Chapter 9 (GUI Applications)
* TBD... 

## Part III - Liveness, Performance, and Testing. 
### Chapter 10 (Avoiding Liveness Hazards)
* TBD...
### Chapter 11 (Performance and Scalability)
* TBD...
### Chapter 12 (Testing Concurrent Programs)
* TBD...

## Part IV - Advanced Topics 

 
