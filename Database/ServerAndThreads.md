# Server and Threads

## Processes, threads, and resource management  

### Processes and threads

#### Differences between Process and Thread
* Thread = a unit of CPI execution + local resources
	* e.g.m program counter, registers, function call stack, etc.
* Process = threads(at least one) + global resources 
	* e.g., memory space/heap, `open files`, etc.

#### Differences between Kernel thread and User thread
* Kernel thread: scheduled by OS. e.g., POSIX Pthreads(UNIX), Win32 threads
* User thead:
	* Scheduled by user applications(in user space above the kernel)
		* Lightweight -> faster to create/destroy
		* Examples: POSIX Pthreads(UNIX), Java threads
	* Eventually mapped to kernel threads
		* Many-to-One
		* One-to-One
		* Many-to-Many

	* Java Thread
		* Scheduled by JVM
		* Mapping depends on the JVM implementation
			* But normally one-to-one mapped to Pthreads/Win32 threads on UNIX/Windows
		* Pros over POSIX(one2one) threads: System independent(if there's a JVM)

### Supporting concurrent clients

#### Serialized or interleaved operations?
* Throughput via Pipelining: Interleaving ops increases throughput by `pipelining CPU and I/O`

#### Statements run by processes or threads?
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

### Embedded Clients
* Running on the same machine as RDBMS
* Usually Single-threaded
	* e.g. sensor nodes, dictionaries, phone apps, etc
* If you need high throughput, manage threads yourself
	* Identify causal relationship between statements
	* Run each group of causal statements in a thread
	* No causal relationship between the results outputted by different groups

### Remote Clients
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

## Implementing JDBC

### RMI
* JDBC Programming
1. Connect to the server
2. Execute the desired query
3. Loop through the result set(for SELECT only)
`4. Close` the connection
	* A result set ties up valuable resource on the server, such as buffers and locks
	* Client should close it connection as soon as the database is no longer needed

#### Implementing JDBC in VanillaCore
* JDBC API is defined `at client side`
* Needs both client- and server-side implementations
	* in org.vanilladb.core.remote.jdbc package
	* JdbcXxx are client-side classes
	* RemoteXxx are server-side classes
* Based on Java RMI
	* Handles server threading: dispatcher thread, worker threads, and thread pool
	* But no control to pool size
	* `Synchronizes` a client thread with a worker thread
		* Blocking method calls at clients

#### Java RMI
* Java RMI allows methods of an object at server VM to be invoked remotely at a client VM
	* We call this object `a remote object`
* How? -> The Stub and Skeleton
```
┌────────────┬──────┐ ─────call─────▻ ┌──────────┬────────────┐
│ RMI Client │ Stub │                 │ Skeleton │ RMI Server │
└────────────┴──────┘ ◅───return───── └──────────┴────────────┘
```

1. The `skeleton` (run by a server thread) binds the interface of the remote object
2. A client thread looks up and obtain a `stub` of the skeleton
3. When a client thread invokes a method, it is blocked and the call is first forwarded to the stub
4. The stub marshals the parameters and  sends the call to the skeleton through the network
5. The skeleton receives the call, unmarshals the parameters, allocates from pool a worker thread that runs the remote object's method on behalf of the client
6. When the method returns, the worker thread returns the result to skeleton and returns to pool
7. The skeleton marshals the results and send it to stub
8. The stub unmarshals the results and continues the client thread

* RMI Registry
	* The server must first bind the remote object's interface to the registry with a name
		* The interface must extend the java.rmi.Remote interface

* Things to Note
	* A client thread and a worker thread is synchronized
	* The same remote object is run by multiple worker threads(each per client)
		`* Remote objects bound to registry must be thread-safe`
	* If the return of a remote method is another remote object, the sub of that object is created automatically and sent back to the client
		* The object can be either thread-local or thread-safe, depending on whether it is created or reused during each method call
	* A remote object will `not` be garbage collected if there's a client holding its stub
		* Destroy stub(e.g., closing connection) at client side ASAP

### Remote Interfaces and client-side wrappers
* Server-Side JDBC Impl
	* RemoteXxx classes that mirror their corresponding `JDBC interfaces` at client-side
		* Implement the most essential JDBC method only
	* Interfaces: RemoteDriver, RemoteConnection, RemoteStatement, RemoteResultSet and RemoteMetaData
		* To be bound to registry
		* Extend java,rmi.Remote
		* Throw RemoteException instead of SQLException 

* Registering Remote Objects
	* Only the RemoteDriver need to be bound to registry
		* Stubs of other can be obtained by method returns
	* Done by `JdbcStartUp`:
		```
		/* create a registry specific for the server on the default port 1099 */
		Registry reg = LocateRegistry.createRegistry(1099);
		
		// post the server entry in it
		RemoteDriver d = new RemoteDriverImpl();

		/* create a stub for the remote implementation object d, save it in the RMI registry */
		reg.rebind("vanilladb-jdbc", d);
		```		

* Obtaining Stubs
	* To obtain the stubs at client-side:
		```
		// url = "jdbc:vanilladb://xxx.xxx.xxx.xxx:1099"
		String host = url.replace("jdbc:vanilladb://", "");
		Registry reg = LocateRegistry.getRegistry(host);
		RemoteDriver rdvr = (RemoteDriver)reg.lookup("vanilladb-jdbc");

		// creates connection
		RemoteConnection rconn = rdvr.connect();
		// creates statement
		RemoteStatement rstmt = rconn.createStatement();
		```
	* Directly through registry or indirectly through method returns

* JDBC Client-Side Impl
	* Implement java.sql interfaces using the client-side `wrappers of stubs`
		* e.g., JdbcDriver wraps the stub of RemoteDriver

### Remote Implementations
* RemoteDriverImpl
	* RemoteDriverImpl is the entry point into the server
	* Each time its connect method is called (via the stub), it creates a new RemoteConnectionImpl on the server
		* RMI creates the corresponding stub and returns back it to the client
	* `Run by multiple threads, must be thread-safe`

* RemoteConnectionImpl
	* Manages client connections on the server
		* Associated with a tx
		*  commit() commits the current tx and starts a new one immediately
	*  `Thread local`

* RemoteStatementImpl
	* Executes SQL statements
		* Creates a planner that finds the best plan tree
	* If the connection is set to be `auto commit`, the executeUpdate() method will call connection.commit() in the end
	* `Thread local`

* RemoteResultSetImpl
	* Provides methods for iterating the output records
		* The scan opened from the best plan tree
	* Tx spans through the iteration
		* Avoid doing heavy jobs during the iteration
	* `Thread local`

* RemoteMetaDataImpl
	* Provides the schema information about the query results
		* Contains the Schema object of the output table
	* `Thread local`

### StartUp
* StartUp provides main() that runs VanillaCore as a JDBC server
	* Calls VanillaDB.init()
		* Sharing global resources through static variables
	* Binds RemoteDriver to RMI registry
		* One thread per connection

### Threading in Engines(Generally)
* Classes in the query engine are thread-local
* Classes in the storage engine are thread-safe

# References
* [Introduction to Database System by Shan-Hung Wu](https://www.youtube.com/watch?v=E4DLGaZGWMk&list=PLS0SUwlYe8cyln89Srqmmlw42CiCBT6Zn&index=15)
