# Java Concurrency In Practice 
## The introduction 
### Chapter 1: Introduction
#### Benefits of Threads
* Exploiting Multiple Processors
* Simplicity of Modeling
* Simplified Handling of Asynchronous Events
* More Responsive User Interfaces

#### Safety Hazards

* e.g., race-condition
#### Liveness Hazards
* A liveness failure occurs when an activity gets into a state such that it is permanently unable to make forward progress. (e.g. inadvertent infinite loop)
* Chapter 10 describes various forms of liveness failures and how to avoid them, 
	* deadlock (Section 10.1), 
	* starvation (Section 10.3.1), 
	* livelock (Section 10.3.3). 
#### Performance Hazards

* e.g., context-switch

## Part I - Fundamentals: 
### Chapter 2. Thread Safety

* This chapter is about writing correct concurrent programs is primarily about managing access to shared, mutable state.

* If multiple threads access the same mutable state variable without appropriate synchronization, your program is
  broken. There are three ways to fix it:
    * Don't share the state variable across threads;
    * Make the state variable immutable; or
    * Use synchronization whenever accessing the state variable

#### 2.1 What is thread-safe?

* https://courses.engr.illinois.edu/cs225/fa2022/resources/stack-heap/

* A class is thread-safe if it behaves correctly when accessed from multiple threads, regardless of the scheduling or
  interleaving of the execution of those threads by the runtime environment, and with no additional synchronization or
  other coordination on the part of the calling code.
* Thread-safe classes encapsulate any needed synchronization so that clients need not provide their own.
* Stateless objects are always thread-safe.
    * Stateless object: Stateless object is an instance of a class without instance fields (instance variables). The
      class may have fields, but they are compile-time constants (static final).
    * Immutable object: Immutable objects may have state, but it will not be change in run-time. e.g., instance
      variables with `final` keyword

#### 2.2 Atomicity

* Atomic: execute as a single, indivisible operation
* Race Conditions
    * What
      is [Race Condition](https://github.com/EddieChoCho/tech_notes/blob/master/OperatingSystem/ProcessSynchronization.md#race-condition):
        * The situation where several threads `access and manipulate shared data concurrently`
        * The final value of the shared data `depends upon which thread finished last`
    * A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of
      multiple threads by the runtime
    * Check-then-act operations (e.g. lazy initialization)
        * You observe something to be true (file X doesn't exist) and then take action based on that observation (create
          X)
        * But in fact the observation could have become invalid between the time you observed it and the time you acted
          on it (someone else created X in the meantime), causing a problem (unexpected exception, overwritten data,
          file corruption).
    * Read-modify-write operations (e.g. increment)
        * Like incrementing a counter, define a transformation of an object's state in terms of its previous state.
    * Race condition v.s. Data race
        * Data race: synchronization is not used to coordinate all access to a shared non-final field.
        * Not all race conditions are data races, and not all data races are race conditions, but they both can cause
          concurrent programs to fail in unpredictable ways.

* Compound Actions
    * We refer collectively to check-then-act and read-modify-write sequences as compound actions: sequences of operations that must be executed atomically in order to remain thread-safe.
    * Use existing thread-safe objects, like AtomicLong, to manage your class's state.        
        
#### 2.3 Locking
* Intrinsic Locks(Monitor Locks)
    * Synchronized block: Java built-in locking mechanism for enforcing atomicity.
    * Every Java object can implicitly act as a lock for purposes of synchronization.
    * Synchronized block has two parts: A reference to an object that will serve as the lock. a block of code to be guarded by that lock.
    * Only one thread at a time can execute a block of code guarded by a given lock.
    * Performance problem
    
* Reentrancy
    * Intrinsic Locks are reentrant.
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

#### 2.5 Liveness and Performance
* There is frequently a tension between simplicity and performance. When implementing a synchronization policy, resist the temptation to prematurely sacrifice simplicity (potentially compromising safety) for the sake of performance.
* Holding a lock for a long time, either because you are doing something compute-intensive or because you execute a potentially blocking operation, introduces the risk of liveness or performance problems.
* Avoid holding locks during lengthy computations or operations at risk of not completing quickly such as network or console I/O.

### Chapter 3. Sharing Objects

* This chapter examines techniques for sharing and publishing objects, so they can be safely accessed by multiple
  threads.
* Together, they lay the foundation for building thread-safe classes and safely structuring concurrent applications
  using the java.util.concurrent library classes.

* Synchronized is not only about atomicity or demarcating "critical sections".
    * Critical sections:
        * Accesses to shared data are encapsulated in regions of code guarded by synchronization primitives (e.g. locks)
          . Such guarded regions of code are called critical sections.
        * The semantics of a critical section dictate that only one thread can execute it at a given time. Any other
          thread that requires access to shared data must wait for the current thread to complete the critical section.

* Synchronization also has another significant, and subtle, aspect: memory visibility.
* We want not only to prevent one thread from modifying the state of an object when another is using it, but also to
  ensure that when a thread modifies the state of an object, other threads can actually see the changes that were made.

#### 3.1 Visibility
* There is no guarantee that the reading thread will see a value written by another thread on a timely basis, or even at all. 
* In order to ensure visibility of memory writes across threads, you must use synchronization.
* There is no guarantee that operations in one thread will be performed in the order given by the program, as long as the reordering is not detectable from within that thread - even if the reordering is apparent to other threads.
* In the absence of synchronization, the Java Memory Model permits the compiler to reorder operations and cache values in registers, and permits CPUs to reorder operations and cache values in processor-specific caches. (see Chapter 16)
* Always use the proper synchronization whenever data is shared across threads.

* Stale Data
    * A thread may see an out-of-date value. Unless synchronization is used every time a variable is accessed, it is possible to see a stale value for that variable.
    * Worse, staleness is not all-or-nothing: a thread can see an up-to-date value of one variable but a stale value of another variable that was written first.
    * Reading data without synchronization is analogous to using the READ_UNCOMMITTED isolation level in a database, where you are willing to trade accuracy for performance. 
        * However, in the case of un-synchronized reads, you are trading away a greater degree of accuracy, since the
          visible value for a shared variable can be arbitrarily stale.
    * Synchronizing only the setter would not be sufficient: threads calling get would still be able to see stale values.
        
* Non-atomic 64-bit Operations
    * Out-of-thin-air safety
        * When a thread reads a variable without synchronization, it may see a stale value, but at least it sees a value that was actually placed there by some thread rather than some random value.
    * Out-of-thin-air safety applies to all variables, with one exception: 64-bit numeric variables (double and long) that are not declared volatile (see Section 3.1.4)
    * The Java Memory Model requires fetch and store operations to be atomic.
    * But for nonvolatile long and double variables, the JVM is permitted to treat a 64-bit read or write as two separate 32-bit operations. 
    * It is not safe to use shared mutable long and double variables in multithreaded programs unless they are declared `volatile` or guarded by a `lock`.
    
* Locking and Visibility
    * Intrinsic locking can be used to guarantee that one thread sees the effects of another in a predictable manner.
    * Locking is not just about mutual exclusion; it is also about memory visibility. To ensure that all threads see the most up-to-date values of shared mutable variables, the reading and writing threads must synchronize on a common lock.
    * Notes from Thinking in Java:
      * Brian’s Rule of Synchronization:
      * If you are writing a variable that might next be read by another thread, or reading a variable that might have
      last been written by another thread, you must use synchronization, and further, both the reader and the writer
      must synchronize using the same monitor lock. * This is an important point: Every method that accesses a critical
      shared resource must be synchronized, or it won’t work right.
            
* Volatile Variables
    * When a field is declared volatile, the compiler and runtime are put on notice that this variable is shared and that operations on it should not be reordered with other memory operations. 
    * Volatile variables are not cached in registers or in caches where they are hidden from other processors.
    * From a memory visibility perspective, writing a volatile variable is like exiting a synchronized block and reading a volatile variable is like entering a synchronized block.
    
    * Practices 
        * However, code that relies on volatile variables for visibility of arbitrary state is more fragile and harder to understand than code that uses locking.
        * Use volatile variables only when they simplify implementing and verifying your synchronization policy; avoid using volatile variables when verifying correctness would require subtle reasoning about visibility. 
        * Good uses of volatile variables include ensuring the visibility of their own state, that of the object they refer to, or indicating that an important lifecycle event (such as initialization or shutdown) has occurred.
        
    * Debugging tip
        * For server applications, be sure to always specify the -server JVM command line switch when invoking the JVM, even for development and testing. The server JVM performs more optimization than the client JVM, such as hoisting variables out of a loop that are not modified in the loop.
        * Code that might appear to work in the development environment (client JVM) can break in the deployment environment (server JVM).
    
    * Limitations of Volatile Variables
        * The semantics of volatile are not strong enough to make the increment operation (count++) atomic.
        * Locking can guarantee both visibility and atomicity; volatile variables can only guarantee visibility.
        
    * Use volatile variables only when all the following criteria are met:
        * Writes to the variable do not depend on its current value, or you can ensure that only a single thread ever updates the value.
        * The variable does not participate in invariants with other state variables; and
        * Locking is not required for any other reason while the variable is being accessed.      

#### 3.2 Publication and Escape
* What is `publishing an object`
    * Making it available to code outside its current scope
        * By storing a reference to it where other code can find it.
        * Returning it from a non-private method.
        * Or passing it to a method in another class.
* Publishing internal state variables can compromise encapsulation and make it more difficult to preserve invariants; (
  ?)
* Publishing objects before they are fully constructed can compromise thread safety
* What is `escape`
    * An object that is published when it should not have been is said to have escaped.
* Any object that is reachable from a published object by following some chain of non-private field references and
  method calls has also been published.
* Once an object escapes, you have to assume that another class or thread may, maliciously or carelessly, misuse it.
* This is a compelling reason to use encapsulation: it makes it practical to analyze programs for correctness and harder
  to violate design constraints accidentally.
* Do not allow the `this` reference to escape during construction.

#### 3.3 Thread Confinement

* Accessing shared, mutable data requires using synchronization; one way to avoid this requirement is to not share.
* If data is only accessed from a single thread, no synchronization is needed. This technique, thread confinement, is
  one of the simplest ways to achieve thread safety.
* When an object is confined to a thread, such usage is automatically thread-safe even if the confined object itself is
  not.
*
    * Comment application of thread confinement
        * The use of pooled JDBC (Java Database Connectivity) Connection objects.
        * Since most requests, such as servlet requests or EJB calls, are processed synchronously by a single thread,
          and the pool will not dispense the same connection to another thread until it has been returned, this pattern
          of connection management implicitly confines the Connection to that thread for the duration of the request.
* The language and core libraries provide mechanisms that can help in maintaining thread confinement - local variables
  and the ThreadLocal class.
* But even with these, it is still the programmer's responsibility to ensure that thread-confined objects do not escape
  from their intended thread.

* Ad-hoc Thread Confinement
    * Ad-hoc thread confinement describes when the responsibility for maintaining thread confinement falls entirely on
      the implementation.

* Stack Confinement(within-thread or thread-local usage)
    * Stack confinement is a special case of thread confinement in which an object can only be reached through local
      variables.
    * It is simpler to maintain and less fragile than ad-hoc thread confinement.

* ThreadLocal
    * A more formal means of maintaining thread confinement is ThreadLocal, which allows you to associate a per-thread
      value with a value-holding object.
    * This technique can also be used when a frequently used operation requires a temporary object such as a buffer and
      wants to avoid reallocating the temporary object on each invocation.
    * Since now days we use threads from the thread pool, a thread can be used by multiple request, we need to clean up
      the ThreadLocal value properly.

#### 3.4 Immutability 
* Immutable objects are always thread-safe.
* Immutable objects are safe to share and publish freely without the need to make defensive copies.
* An object is immutable if:
    * Its state cannot be modified after construction;
    * All its fields are final; (It is technically possible to have an immutable object without all fields being final.
      String is such a class but this relies on delicate reasoning about benign data races that requires a deep
      understanding of the Java Memory Model.)
    * It is properly constructed (the `this` reference does not escape during construction).

* Final Fields
    * Final fields can't be modified (although the objects they refer to can be modified if they are mutable).
    * Final fields also have special semantics under the Java Memory Model. It is the use of final fields that makes possible the guarantee of initialization safety (see Section 3.5.2) that lets immutable objects be freely accessed and shared without synchronization.
    * Just as it is a good practice to make all fields private unless they need greater visibility, it is a good practice to make all fields final unless they need to be mutable.

#### 3.5 Safe Publication
* Immutable Objects and Initialization Safety
    * JavaMemory Model offers a special guarantee of initialization safety for sharing immutable objects. 
    * An object reference becomes visible to another thread does not necessarily mean that the state of that object is visible to the consuming thread. In order to guarantee a consistent view of the object's state, synchronization is needed.
    * Immutable objects(unmodifiable state, all fields are final, and proper construction) can be used safely by any thread without additional synchronization, even when synchronization is not used to publish them.
     
* Safe Publication Idioms
    * To publish an object safely, both the reference to the object and the object's state must be made visible to other
      threads at the same time. A properly constructed object can be safely published by:
        * Initializing an object reference from a static initializer;
        * Storing a reference to it into a volatile field or AtomicReference;
        * Storing a reference to it into a final field of a properly constructed object; or
        * Storing a reference to it into a field that is properly guarded by a lock.
    * The thread-safe library collections offer the following safe publication guarantees:
        * Placing a key or value in a Hashtable, synchronizedMap, or Concurrent-Map safely publishes it to any thread
          that retrieves it from the Map (whether directly or via an iterator);
        * Placing an element in a Vector, CopyOnWriteArrayList, CopyOnWrite-ArraySet, synchronizedList, or
          synchronizedSet safely publishes it to any thread that retrieves it from the collection;
        * Placing an element on a BlockingQueue or a ConcurrentLinkedQueue safely publishes it to any thread that
          retrieves it from the queue.

* Effectively Immutable Objects
    * Safely published effectively immutable objects can be used safely by any thread without additional synchronization.

* Mutable Objects
    * The publication requirements for an object depend on its mutability:
        * Immutable objects can be published through any mechanism;
        * Effectively immutable objects must be safely published;
        * Mutable objects must be safely published, and must be either thread-safe or guarded by a lock

* Sharing Objects Safely

### Chapter 4. (Composing Objects)

* This chapter covers patterns for structuring classes that can make it easier to make them thread-safe and to maintain
  them without accidentally undermining their safety guarantees.

#### 4.1. Designing a Thread-safe Class

* The design process for a thread-safe class should include these three basic elements:
    * Identify the variables that form the object's state;
    * Identify the invariants that constrain the state variables;
    * Establish a policy for managing concurrent access to the object's state.

* Synchronization policy
    * The synchronization policy defines how an object coordinates access to its state without violating its invariants
      or postͲ conditions.
    * It specifies what combination of immutability, thread confinement, and locking is used to maintain thread safety,
      and which variables are guarded by which locks.
    * To ensure that the class can be analyzed and maintained, document the synchronization policy.

* 4.1.1. Gathering Synchronization Requirements
    * You cannot ensure thread safety without understanding an object's invariants and
      post-conditions(https://en.wikipedia.org/wiki/Postcondition).
    * Constraints on the valid values or state transitions for state variables can create atomicity and encapsulation
      requirements.
* 4.1.2. State-dependent Operations
    * Class invariants and method post-conditions constrain the valid states and state transitions for an object. Some
      objects also have methods with state-based preconditions.
    * Concurrent programs add the possibility of waiting until the precondition becomes true, and then proceeding with
      the operation.
    * The built-in mechanisms for efficiently waiting for a condition to become true - wait and notify - are tightly
      bound to intrinsic locking, and can be difficult to use correctly.

#### 4.2. Instance Confinement

* If an object is not thread-safe, several techniques can still let it be used safely in a multithreaded program.
    * You can ensure that it is only accessed from a single thread (thread confinement).
    * Or that all access to it is properly guarded by a lock.
* Encapsulation simplifies making classes thread-safe by promoting instance confinement, often just called confinement.
* Combining confinement with an appropriate locking discipline can ensure that otherwise non-thread-safe objects are
  used in a thread-safe manner.
* Encapsulating data within an object confines access to the data to the object's methods, making it easier to ensure
  that the data is always accessed with the appropriate lock held.
* Monitor pattern
    * Monitor pattern is used to enforce single-threaded access to data. Only one thread at a time is allowed to execute
      code within the monitor object.
* Thread safety also could be maintained by copying mutable data before returning it to the client.

#### 4.3. Delegating Thread Safety

* Delegation
    * If the components of a class are already thread-safe which makes the class also thread-safe, We could say that the
      class `delegates` its thread safety responsibilities to the components.
    * UnmodifiableMap
        * An unmodifiable view of the specified map.
        * Query operations on the returned map "read through" to the specified map, and attempts to modify the returned
          map, whether direct or via its collection views, result in an UnsupportedOperationException.
* When Delegation Fails
    * If a class has compound actions, delegation alone is not a suitable approach for thread safety.
    * In these cases, the class must provide its own locking to ensure that compound actions are atomic, unless the
      entire compound action can also be delegated to the underlying state variables.

#### 4.4. Adding Functionality to Existing Thread-safe Classes

* Modify the original class to support the desired operation.
    * You need to understand the implementation's synchronization policy so that you can enhance it in a manner
      consistent with its original design.
    * Adding the new method directly to the class means that all the code that implements the synchronization policy for
      that class is still contained in one source file, facilitating easier comprehension and maintenance.

* Extend the class
    * TDB...

#### 4.5. Documenting Synchronization Policies

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
* Two major categories of failed cases - Safety and Liveness
#### Safety - "nothing bad ever happens" 
* e.g. test with cache size linked list - safety test would be to compare the cached count against the actual number of elements in the list.
* This can be done by locking the list for exclusive access
	* Employing some sort of "atomic snapshot" feature provided by the implementation,
	* or by using "test points" provided by the implementation that let tests assert invariants or execute test code atomically.
* Unfortunately, test code can introduce timing or synchronization artifacts that can mask bugs that might otherwise manifest themselves.	
	* Bugs that disappear when you add debugging or test code are playfully called Heisenbugs

#### Liveness - "something good eventually happens"
* Liveness tests include tests of progress and nonͲprogress

* Performance tests. Performance can be measured in a number of ways, including:
	* Throughput: the rate at which a set of concurrent tasks is completed;
	* Responsiveness: the delay between a request for and completion of some action (also called latency); or
	* Scalability: the improvement in throughput (or lack thereof) as more resources (usually CPUs) are made available.

## Part IV - Advanced Topics 

 
