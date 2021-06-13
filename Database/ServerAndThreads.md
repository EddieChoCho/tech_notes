# Server and Threads
## Serialized or interleaved operations?
* Throughput via Pipelining: Interleaving ops increases throughput by `pipelining CPU and I/O`

## Statements run by processes or threads?
* DBMS is about resource management
	* Opened files
	* Buffers(to cache pages)
	* Logs 
	* Locks of objects(incl. files/blocks/record locks)
	* Metadata

* If statements are run by process, then we need inter-process communications(IPC)
	* When, e.g., two statements access the same table(file)
	* System dependent
* Threads allows global resource to be shared directly
	* e.g. through argument passing or `static variables`

* Example: VanillaDb
	* Provides access to global resources through Singleton Instances:
		* FileMgr, BufferMgr, LogMgr, CatalogMgr
	* Creates the new objects that access global resources:
		* Planner and Transaction
	* Before using the VanillaCore, the ```VanillaDb.init(name)``` must be called
		* Initialize file, log, buffer, metadata, and tx mgrs
		* Create or recover the specified database

## Embedded Clients
* Running on the same machine as RDBMS
* Usually Single-threaded
	* e.g. sensor nodes, dictionaries, phone apps, etc
* If you need high throughput, manage threads yourself
	* Identify causal relationship between statements
	* Run each group of causal statements in a thread
	* No causal relationship between the results outputted by different groups

## Remote Clients
* Server has a server/dispatcher thread which keeps listening for client requests
* If the server thread get a client request, it will create a new thread(worker thread) to service the request
* One worker thread per request
* Then the server thread will resume listening for additional client requests

### What is a request? Request == Connection
* In VanillaDB, a worker thread handles all statements issues by the same user
* Rationale:
	* Statements issued by a user are usually in a causal order -> ensure casualty in a session
	* A user may re-examine the data he/she accessed -> easier caching
* Implications:
	* All statements issued in a JDBC connection is run by a single thread at server
	* Number of connections == Number of threads

### Thread Pooling
* Creating/destroying a thread each time upon connection/disconnection leads to large overhead
* To reduce this overhead, a worker `thread pool` is commonly used
	* Threads are allocated from the pool as needed, and returned to the pool when no longer needed
	* When no threads are available in the pool, the client may have to wait until one becomes available
* Graceful performance degradation by limiting the pool size

# References
* [Introduction to Database System by Shan-Hung Wu](https://www.youtube.com/watch?v=E4DLGaZGWMk&list=PLS0SUwlYe8cyln89Srqmmlw42CiCBT6Zn&index=15)
