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


## References
* [Thinking in Java](https://www.amazon.com/Thinking-Java-4th-Bruce-Eckel/dp/0131872486)
