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
    * Check-then-act operations (e.g. lazy initialization)
        * You observe something to be true (file X doesn't exist) and then take action based on that observation (create X)
        * But in fact the observation could have become invalid between the time you observed it and the time you acted on it (someone else created X in the meantime), causing a problem (unexpected exception, overwritten data, file corruption).
    * Read-modify-write operations (e.g. increment)
        * Like incrementing a counter, define a transformation of an object's state in terms of its previous state.    
    * Race condition v.s. Data race
        * Data race: synchronization is not used to coordinate all access to a shared non-final field.
        * Not all race conditions are data races, and not all data races are race conditions, but they both can cause concurrent programs to fail in unpredictable ways.

* Compound Actions
    * We refer collectively to check-then-act and read-modify-write sequences as compound actions: sequences of operations that must be executed atomically in order to remain thread-safe.
    * Use existing threadͲsafe objects, like AtomicLong, to manage your class's state.        
        
#### 2.3 Locking
* Intrinsic Locks(Monitor Locks)
    * Synchronized block: Java built-in locking mechanism for enforcing atomicity.
    * Every Java object can implicitly act as a lock for purposes of synchronization.
    * Synchronized block has two parts: A reference to an object that will serve as the lock. a block of code to be guarded by that lock.
    * Only one thread at a time can execute a block of code guarded by a given lock.
    * Performance problem
    
* Reentrancy
    * Reentrancy means that locks are acquired on a per-thread rather than per-invocation basis.
    * Reentrancy is implemented by associating with each lock an acquisition count and an owning thread. When the count is zero, the lock is considered unheld.
        * When a thread acquires a previously unheld lock, the JVM records the owner and sets the acquisition count to one. 
        * If that same thread acquires the lock again, the count is incremented, and when the owning thread exits the synchronized block, the count is decremented. 
        * When the count reaches zero, the lock is released.
    * Default locking behavior for pthreads (POSIX threads) mutexes, which are granted on a pe-invocation basis.
    * Code that would Deadlock if Intrinsic Locks were Not Reentrant
    ```
    public class Widget {
        public synchronized void doSomething() {
        ...
        }
    }
    public class LoggingWidget extends Widget {
        public synchronized void doSomething() {
            System.out.println(toString() + ": calling doSomething");
            super.doSomething();
     }

    ```
#### 2.4 Guarding State with Locks
* For each mutable state variable that may be accessed by more than one thread, all accesses to that variable must be performed with the same lock held. 
    * In this case, we say that the variable is guarded by that lock.

* Every shared, mutable variable should be guarded by exactly one lock. Make it clear to maintainers which lock that is.
* A common locking convention is to encapsulate all mutable state within an object and to protect it from concurrent access by synchronizing any code path that accesses mutable state using the object's intrinsic lock. (e.g. Vector)
* For every invariant that involves more than one variable, all the variables involved in that invariant must be guarded by the same lock.
* While synchronized methods can make individual operations atomic, additional locking is required - when multiple operations are combined into a compound action.

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

 
