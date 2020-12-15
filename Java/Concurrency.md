### Basic keywords

* Process
    * A process is a self-contained program running within its own memory space.
    * A multitasking operating system can run more than one process (program) at a time by periodically switching the CPU from one process to another, while making it look as if each process is chugging along on its own.

* Heap
    * Each java application gets it’s own memory space (heap).
    * Instance variables

* Thread
    * A thread is a unit of execution within a process. Each process can have multiple threads. Every thread created by a process shares the process's memory and files.

* Thread stack
    * Thread stack is the memory that only that thread can access
    * Local variables
    * [The call stack](https://youtu.be/5xUDoKkmuyw)

### Blocking
* If one task in your program is unable to continue because of some condition outside of the control of the program (typically I/O), we say that the task or the thread blocks. 
* Without concurrency, the whole program comes to a stop until the external condition changes. If the program is written using concurrency, however, the other tasks in the program can continue to execute when one task is blocked, so the program continues to move forward.

#### multiprocessor machine
* multiple tasks can be distributed across those processors, which can dramatically improve throughput.

#### single processor:
* concurrent program running on a single processor should actually have more overhead than if all the parts of the program ran sequentially, because of the added cost of the so-called `context switch` (changing from one task to another).
* From a performance standpoint, it makes no sense to use concurrency on a single-processor machine unless one of the tasks might block.
* Example of performance improvements in single-processor systems: event-driven programming.

### Atomicity
* Atomic operations do not need to be synchronized.
* An atomic operation is one that cannot be interrupted by the thread scheduler; if the operation begins, then it will run to completion before the possibility of a context switch. 
* You do get atomicity (for simple assignments and returns) if you use the volatile keyword when defining a long or double variable (note that volatile was not working properly before Java SE5). 
* Expert programmers can take advantage of this to write lock-free code, which does not need to be synchronized. But trying to remove synchronization is usually a sign of premature optimization, and will cause you a lot of trouble, probably without gaining much, or anything.

#### Atomicity still needs synchronize - both the reader and the writer must synchronize:
* Even an operation is atomic, without the synchronization, the operation still allows to get the object while the object is in an unstable intermediate state.

### Visibility
* Changes made by one task, even if they’re atomic in the sense of not being interruptible, might not be visible to other tasks (the changes might be temporarily stored in a local processor cache, for example).
*  The `synchronization` mechanism forces changes by one task on a multiprocessor system to be visible across the application.

### synchronized
* We should minimize the scope of synchronization for performance reason but also keep the correctness of the code at the same time.
* e.g. We could synchronized on the canonical representation for the string object:
```java
public class Component{
    Map<Sting, Obejct> dataMap = new HasMap<>();
    
    public Object getData(String key){
        Object result = dataMap.get(key);
        if(result == null){
            synchronized(key.intern()){
                if(result == null){
                    //initial data...
                }
            }
        }
        return result;
    }
}
```   

### volatile:
* The volatile keyword also ensures visibility across the application. If you declare a field to be volatile, this means that as soon as a write occurs for that field, all reads will see the change.(even if local caches are involved)
* This is true even if local caches are involved—volatile fields are immediately written through to main memory, and reads occur from main memory.(and are not cached).
* Volatile also restricts compiler reordering of accesses during optimization.

#### Volatile vs Synchronization:
* If multiple tasks are accessing a field, that field should be volatile; otherwise, the field should only be accessed via synchronization.
* Synchronization also causes flushing to main memory, so if a field is completely guarded by synchronized methods or blocks, it is not necessary to make it volatile.

#### Scenarios that volatile won’t work:
* When the value of a field depends on its previous value (such as incrementing a counter).
* TBD

#### Volatile still needs synchronize
* Volatile does not affect the fact that an increment isn’t an atomic operation.

### Thread local storage
* ThreadLocal guarantees that no race condition can occur. individual threads are each allocated their own storage.

### Thread pool
* ForkjoinPool

* FixedThreadPool
    * When all threads are busy, then the executor will queue new tasks.  This way, we have more control over our program's resource consumption.
    * Fixed thread pools are better suited for tasks with unpredictable execution times.

* CachedThreadPool
    * Cached thread pools are using “synchronous handoff” to queue new tasks. 
    * The basic idea of synchronous handoff is simple and yet counter-intuitive: One can queue an item if and only if another thread takes that item at the same time. In other words, the SynchronousQueue can not hold any tasks whatsoever.
    * Suppose a new task comes in. If there is an idle thread waiting on the queue, then the task producer hands off the task to that thread. Otherwise, since the queue is always full, the executor creates a new thread to handle that task.
    * The cached pool starts with zero threads and can potentially grow to have Integer.MAX_VALUE threads. Practically, the only limitation for a cached thread pool is the available system resources.
    * To better manage system resources, cached thread pools will remove threads that remain idle for one minute.
    * Use Cases:
        * It works best when we're dealing with a reasonable number of short-lived tasks. 
        * We should avoid this thread pool when the execution time is unpredictable, like IO-bound tasks.

## ExecutorService[2]
* ExecutorService is a framework provided by the JDK which simplifies the execution of tasks in asynchronous mode. 
* Generally speaking, ExecutorService automatically provides a pool of threads and API for assigning tasks to it.

### Instantiating ExecutorService
*  Factory methods of the Executors class.
	```
	ExecutorService executor = Executors.newFixedThreadPool(10);
	```
* Directly Create an ExecutorService
	```
	ExecutorService executorService = 
  	new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS,   
  	new LinkedBlockingQueue<Runnable>());
	```

### Assigning Tasks to the ExecutorService
* execute()
	* The execute() method is void, and it doesn't give any possibility to get the result of task's execution or to check the task's status (is it running or executed).
* submit() 
	* submit() submits a Callable or a Runnable task to an ExecutorService and returns a result of type Future.
* invokeAny()
	* invokeAny() assigns a collection of tasks to an ExecutorService, causing each to be executed, and returns the result of a successful execution of one task (if there was a successful execution).
* invokeAll()
	* invokeAll() assigns a collection of tasks to an ExecutorService, causing each to be executed, and returns the result of all task executions in the form of a list of objects of type Future.

### Shutting Down an ExecutorService
* In general, the ExecutorService will not be automatically destroyed when there is no task to process. It will stay alive and wait for new work to do.
* This is very helpful if
	* An app needs to process tasks that appear on an irregular basis.
	* The quantity of these tasks is not known at compile time.
* On the other hand, an app could reach its end, but it will not be stopped because a waiting ExecutorService will cause the JVM to keep running.
	* To properly shut down an ExecutorService, we have the shutdown() and shutdownNow() APIs.
* One good way to shut down the ExecutorService (which is also recommended by Oracle) is to use both of these methods combined with the awaitTermination() method. 	
	* With this approach, the ExecutorService will first stop taking new tasks, then wait up to a specified period of time for all tasks to be completed. If that time expires, the execution is stopped immediately.
	```
	executorService.shutdown();
	try {
    	if (!executorService.awaitTermination(800, TimeUnit.MILLISECONDS)) {
        	executorService.shutdownNow();
    	} 
	} catch (InterruptedException e) {
    	executorService.shutdownNow();
	}
	```
### The Future Interface
* The submit() and invokeAll() methods return an object or a collection of objects of type Future, which allows us to get the result of a task's execution or to check the task's status (is it running or executed).

* get()
	* The Future interface provides a special blocking method get() which returns an actual result of the Callable task's execution or null in the case of Runnable task.
	* Calling the get() method while the task is still running will cause execution to block until the task is properly executed and the result is available.
	```
	Future<String> future = executorService.submit(callableTask);
	String result = null;
	try {
    	result = future.get();
	} catch (InterruptedException | ExecutionException e) {
    	e.printStackTrace();
	}
	```
* Timeout
	* With very long blocking caused by the get() method, an application's performance can degrade. If the resulting data is not crucial, it is possible to avoid 
	such a problem by using timeouts. (String result = future.get(200, TimeUnit.MILLISECONDS);)

* isDone()
	* The isDone() method can be used to check if the assigned task is already processed or not.

* cancel() and isCancelled()
	* The Future interface also provides for the cancellation of task execution with the cancel() method, and to check the cancellation with isCancelled() method.

### The ScheduledExecutorService Interface
* The ScheduledExecutorService runs tasks after some predefined delay and/or periodically. 
* We can use the factory methods of the Executors class to instantiate a ScheduledExecutorService.

* scheduled()
	* Schedule a single task's execution after a fixed delay. (e.g. Future<String> resultFuture = executorService.schedule(callableTask, 1, TimeUnit.SECONDS))

* scheduleAtFixedRate() 
	* Execute a task periodically after a fixed delay. 
	* Period execution of the task will end at the termination of the ExecutorService or if an exception is thrown during task execution.
* scheduleWithFixedDelay()
	* Have a fixed length delay between iterations of the task.
	* Period execution of the task will end at the termination of the ExecutorService or if an exception is thrown during task execution.
### ExecutorService vs. Fork/Join
* The best use case for ExecutorService is the processing of independent tasks, such as transactions or requests according to the scheme “one thread for one task.”
* In contrast, [according to Oracle's documentation](https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html), fork/join was designed to speed up work which can be broken into smaller pieces recursively.

### Beware
* Wrong thread-pool capacity while using fixed length thread-pool
	* It is very important to determine how many threads the application will need to execute tasks efficiently. 
	* A thread-pool that is too large will cause unnecessary overhead just to create threads which mostly will be in the waiting mode. 
	* Too few can make an application seem unresponsive because of long waiting periods for tasks in the queue.
* Calling a Future‘s get() method after task cancellation.
	* An attempt to get the result of an already canceled task will trigger a CancellationException.
* Unexpectedly-long blocking with Future‘s get() method.
	* Timeouts should be used to avoid unexpected waits.

## Guide To CompletableFuture[3]

### Running Multiple Futures in Parallel
* CompletableFuture.allOf: The CompletableFuture.allOf static method allows to wait for completion of all of the Futures provided as a var-arg.
```
CompletableFuture<Void> combinedFuture = CompletableFuture.allOf(future1, future2, future3);
combinedFuture.get();
```

* CompletableFuture.join(): It returns the combined results of all Futures.
```
String combined = Stream.of(future1, future2, future3)
  .map(CompletableFuture::join)
  .collect(Collectors.joining(" "));
```

## References
* [1][Thinking in Java](https://www.amazon.com/Thinking-Java-4th-Bruce-Eckel/dp/0131872486)
* [2][A Guide to the Java ExecutorService | Baeldung](https://www.baeldung.com/java-executor-service-tutorial)
* [3][Guide To CompletableFuture | Baeldung](https://www.baeldung.com/java-completablefuture)
