# Cats Effect
## Concurrency Basics
### Dictionary
* Parallelism
	* Using multiple computational resources (like more processor cores) to perform a computation faster, usually executing at the same time.
	* Example: summing a list of Integers by dividing it in half and calculating both halves in parallel.
	* Main concern: efficiency. 

* Concurrency
	* Multiple tasks interleaved. Concurrency doesn’t have to be multithreaded. We can write concurrent applications on single processor using methods such as event loops.
	* Example: Communicating with external services through HTTP.
	* Main concern: interaction with multiple, independent and external agents.

* CPU-bound task
* IO-bound task
* Non-terminating task

#### Threads
* Threading (on JVM)
	* Threads in JVM map 1:1 to the operating system’s native threads. 
	* If we try to run too many threads at once we will suffer because of many context switches. 
	* Before any thread can start doing real work, the OS needs to store state of earlier task and restore the state for the current one. This cleanup has nontrivial cost. 
	* The most efficient situation for CPU-bound tasks is when we execute as many threads as the number of available cores because we can avoid this overhead. Therefore, synchronous execution can have better throughput than parallel execution. 
	* If you parallelize it too much, it won’t make your code magically faster. The overhead of creating or switching threads is often greater than the speedup, so make sure to benchmark.
	* Remember that threads are scarce resource on JVM. If you exploit them at every opportunity it may turn out that your most performance critical parts of the application suffer because the other part is doing a lot of work in parallel, taking precious native threads.

* Thread Pools
	* Creating a Thread has a price to it. The overhead depends on the specific JVM and OS, but it involves making too many threads for short-lived tasks is very inefficient. 
	* It may turn out that process of creating thread and possible context switches has higher costs than the task itself. Furthermore, having too many threads means that we can eventually run out of memory and that they are competing for CPU, slowing down the entire application.
	* It is advised to use thread pools created from java.util.concurrent.Executor. A thread pool consists of work queue and a pool of running threads. 
	* Every task (Runnable) to execute is placed in the work queue and the threads that are governed by the pool take it from there to do their work. 
	* In Scala, we avoid explicitly working with Runnable and use abstractions that do that under the hood (Future and IO implementations). 
	* Thread pools can reuse and cache threads to prevent some of the problems mentioned earlier.

##### Choosing Thread Pool
* Thread pool best practices
	* Computation: work-stealing, CPU-bound, Finite resources, avoid blocking at all coasts
	* Event Dispatcher: Highest priority 10 couple of threads, avoid work at all cost
	* Blocking IO: Caching unbounded size

* Bounded
	* Limiting number of available threads to certain amount. Example could be newSingleThreadExecutor to execute only one task at the time or limiting number of threads to number of processor cores for CPU-bound tasks.

* Unbounded
	* No maximum limit of available threads. Note that this is dangerous because we could run out of memory by creating too many threads, so it’s important to use cached pool (allowing to reuse existing threads) with keepalive time (to remove useless threads) and control number of tasks to execute by other means (backpressure, rate limiters).
	* Despite those dangers it is still very useful for blocking tasks. In limited thread pool if we block too many threads which are waiting for callback from other (blocked) thread for a long time we risk getting deadlock that prevents any new tasks from starting their work.

* Blocking Threads
	* As a rule we should never block threads, but sometimes we have to work with interface that does it. 
	* Blocking a thread means that it is being wasted and nothing else can be scheduled to run on it. As mentioned, this can be very dangerous and it’s best to use dedicated thread pool for blocking operations. This way they won’t interfere with CPU-bound part of application.

* Green Threads
    * There are more types of threads and they depend on the platform. One of them is green thread.
    * Green threads are threads that are scheduled by a runtime library or virtual machine (VM) instead of natively by the underlying operating system (OS).
    * The main difference between model represented by JVM Threads and Green Threads is that the latter are not scheduled on OS level.
    * Green threads emulate multithreaded environments without relying on any native OS abilities, and they are managed in user space instead of kernel space, enabling them to work in environments that do not have native thread support.
    * They are often characterized by cooperative multitasking which means the thread decides when it’s giving up control instead of being forcefully preempted, as happens on the JVM. 
    * This term is important for Cats Effect, whose Fiber and shift design have a lot of similarities with this model.

#### Thread Scheduling
* IO.shift function(cats.effect.IO)
    * This function allows to shift computation to different thread pool.
    * This function can introducing `Asynchronous Boundary`: it send computation to current ExecutionContext to schedule it again.

* Thread scheduling - The Essential term, what is happening behind the scenes during shift.
    * Since we can’t run all our threads in parallel all the time, they each get their own slice of time to execute, interleaving with the rest of them so every thread has a chance to run. 
    * When it is time to change threads, the currently running thread is preempted. It saves its state and the context switch happens.
    * A bit different when using thread pools (ExecutionContexts): 
        * They are in charge of scheduling threads from their own pool.
        * If there is one thread running, it won’t change until it terminates or higher priority thread is ready to start doing work.
    * Note that IO without any shifts is considered one task.
        If it’s infinite IO, it could hog the thread forever and if we use single threaded pool, nothing else will ever run on it!
        * `IO` is executing synchronously until we call `IO.shift` or use function like `parSequence`.
    * In terms of individual thread pools, we can actually treat `IO` like green thread with `cooperative multitasking`. 
    * Instead of preemption, we can decide when we yield to other threads from the same pool by calling `shift`.        
           
 * TBD....

### Futher Readings 
* [Practical FP in Scala: A hands-on approach(cat-effect and related libs)](https://leanpub.com/pfp-scala)
* [Daniel Spiewak’s gist](https://gist.github.com/djspiewak/46b543800958cf61af6efa8e072bfd5c)