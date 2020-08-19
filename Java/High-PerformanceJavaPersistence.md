# High-Performance Java Persistence
## I.Introduction 
### 1.Preface
### 2.Performance and Scaling
#### 2.1 Response time and throughput
##### Transaction response time
* The transaction response time is measured as the time it takes to complete a transaction.
* It encompasses the following time segments:
    * The database connection acquisition time 
    * The time it takes to send all database statements over the wire
    * The execution time for all incoming statements
    * The time it takes for sending the result sets back to the database client
    * The time the transaction is idle due to application-level computations prior to releasing the database connection.
    
##### Throughput
* Throughput is defined as the rate of completing incoming load. In a database context.
* Throughput can be calculated as the number of transactions executed within a given time interval.
* Throughput = transaction count / time

##### Response time and throughput
* Ideally, if the system was scaling linearly, adding more database connections would yield a proportional throughput increase. 
* Due to contention on database resources and the cost of maintaining coherency across multiple concurrent database sessions, the relative throughput gain follows a curve instead of a straight line.
* C(N) = N / 1 + α (N − 1) + βN (N − 1)
    * C - the relative throughput gain for the given concurrency level
    * α - the contention coefficient (the serializable portion of the data processing routine)
    * β - the coherency coefficient (the cost for maintaining consistency across all concurrent database sessions).
    * N - the number of concurrent sessions.

#### 2.2 Database connections boundaries
* The total number of connections(TCP sockets from the client (application) to the server (database).) offered by a database server depends on the underlying hardware resources
* Finding how many connections a server can handle is possible through measurements and proven scalability models.
* Thread-based connection handling: SQL Server 2016 and MySQL 5.7
* One operating system process for each individual connection: PostgreSQL 9.5
* Oracle
    * On Windows systems, Oracle uses threads, while on Linux, it uses process-based connections.
    * Oracle 12c comes with a thread-based connection model for Linux systems too.
* High-throughput database applications will experience contention on CPU, Memory, Disk and Locks.
* CPU, Memory, Disk: Even if all indexes are entirely cached in memory, disk access might still occur (to fetch the associated data pages into the memory buffer pool). 
* Locks: To provide data consistency, locks (shared and exclusive) are used to protect data blocks (rows and indexes) from being concurrently updated.

* The first step to improving a system throughput: to tune it according to the current data access patterns.
    * Resources might get saturated due to improper system configuration.
    
#### 2.3 Scaling up and scaling out
* Vertical scaling  (scaling up): Adding resources to a single machine. 
* Horizontal scaling (scaling out): Increasing the number of available machines.
* Database replication:
    * Eliminating the single point of failure.
    * Increase transaction throughput. (In a Master-Worker topology, the Worker nodes can accept read-only transactions, therefore routing read traffic away from the Master node.)
##### 2.3.1 Master-Worker replication
* The Master is the system of record and the only node accepting writes. 
* All changes recorded by the Master node are replayed onto Workers as well.
* The Worker nodes increase the available read-only connections and reduce Master node resource contention, which, in turn, can also lower read-write transaction response time. 
    * Binary replication: uses the Master node WAL (Write Ahead Log).
    * Statement-based replication: replays on the Worker machines the exact statements executed on Master.
    * Asynchronous replication: 
        * It is very common, especially when there are more Worker nodes to update.
        * The Worker nodes are eventual consistent as they might lag behind the Master. 
        * In case the Master node crashes, a cluster-wide voting process must elect the new Master (usually the node with the most recent update record) from the list of all available Workers.
        * Warm standby: asynchronous replication topology
        * The new elected Worker node might lag behind the failed Master, in which case consistency and durability are traded for lower latencies and higher throughput.

    * Having one synchronous Worker node
        * It allows the system to ensure data consistency in case of Master node failures since the synchronous Worker is an exact copy of the Master.
        * Most database systems allow one synchronous Worker node, at the price of increasing transaction response time (the Master has to block waiting for the synchronous Worker node to acknowledge the update). 
        * In case of Master node failure, the automatic failover mechanism can promote the synchronous Worker node to become the new Master
        * Hot standby:  synchronous Master-Worker replication
##### 2.3.2 Multi-Master replication
* If the Master node can no longer keep up with the ever increasing read-write traffic, a Multi-Master replication might be a better alternative.
* Trade-offs of distributed systems
    * Ensuring data consistency is challenging in a Multi-Master replication scheme because there is no longer a single source of truth.
    * The same data can be modified concurrently on separate nodes, so there is a possibility of conflicting updates. 
    * Automatic conflict resolution algorithm: Avoid or detect conflicts.
* Two-phase commit protocol:
    * To enlist all participating nodes in one distributed transaction. 
    * This design allows all nodes to be in-sync at all time, at the cost of increasing transaction response time (by slowing down write operations).
    * If nodes are separated by a WAN (Wide Area Network), synchronization latencies can increase dramatically.
* The asynchronous Multi-Master replication
    * It requires a conflict detection and an automatic conflict resolution algorithm. 
    * When a conflict is detected, the automatic resolution tries to merge the two conflicting branches.
    * In case it fails, manual intervention is required.

##### 2.3.3 Sharding
* Horizontal partitioning
    * Traditionally, relational databases have offered horizontal partitioning to distribute data across multiple tables within the same database server. 
* Sharding 
    * Sharding means distributing data across multiple nodes so each instance contains only a subset of the overall data.(As opposed to horizontal partitioning.)
    * Each shard must be self-contained because a user transaction can only use data from within a single shard. 
    * Joining across shards is usually prohibited because the cost of distributed locking and the networking overhead would lead to long transaction response times.
    * By reducing data size per node, indexes will also require less space, and they can better fit into main memory. 
    * With less data to query, the transaction response time can also get shorter too.

* Sharding is usually a last resort strategy, employed after exhausting all other available options, such as:
    * Optimizing the data layer to deliver lower transaction response times
    * Scaling each replicated node to a cost-effective configuration
    * Adding more replicated nodes until synchronization latencies start dropping below an acceptable threshold.

* MySQL cluster auto-sharding
    * The parts similar to the Multi-Master replication architecture: 
        * Every node accepts both read and write transactions and, just like Multi-Master replication, conflicts are automatically discovered and resolved.
        * Auto-sharding can increase throughput by distributing incoming load to multiple machines.
         
    * The parts different to the Multi-Master replication architecture:     
        While in a Multi-Master replicated environment every node stores the whole database, the auto-sharding cluster distributes data so that each shard is only a subset of the whole database.

## II.JDBC and Database Essentials
### 3.JDBC Connection Management
* The java.sql.Driver
    * The java.sql.Driver is the main entry point for interacting with the JDBC API, defining the implementation version details and providing access to a database connection.
    * JDBC defines four driver types(Being easier to setup and debug, the Type 4 driver is usually the preferred alternative.): 
        * Type 1: It’s only a bridge to an actual ODBC driver implementation
        * Type 2: It uses a database specific native client implementation (e.g. Oracle Call Interface)
        * Type 3: It delegates calls to an application server offering database connectivity support
        * Type 4: The JDBC driver implements the database communication protocol solely in Java.

* java.sql.Connection
    * To communicate to a database server, a Java program must first obtain a java.sql.Connection
    
##### 3.1 DriverManager
* getConnection()
    * Every time the getConnection() method is called, the DriverManager will request a new physical connection from the underlying Driver.
    * Application -> DriverManager -> Driver -> Connection -> SocketFactory -> Socket -> Database
    * Application <- DriverManager <- Driver <- Connection <- SocketFactory <- Socket <- Database
    * Application          -(close)->           Connection      -(close)->     Socket
* Two-tier architecture
    * The application is run by single user, and each instance uses a dedicated database connection. 
    * The more users, the more database connections are required.

##### 3.2 DataSource
* DriverManager is a physical connection factory, DataSource interface is a logical connection provider
* Three-tier architecture
* Advantages:
    * As a database connection buffer
		* The user request throughput is greater than the available database connection capacity. As long as the connection acquisition time is tolerable (from the end-user perspective), the user request can wait for a database connection to become available. 
		* The middle layer acts as a database connection buffer that can mitigate user request traffic spikes by increasing request response time, without depleting database connections or discarding incoming traffic.
	* Monitor connection
		* Because the intermediate layer manages database connections, the application server can also monitor connection usage and provide statistics to the operations team.
		* For this reason, instead of serving physical database connections, the application server provides only logical connections (proxies or handles), so it can intercept and register how the client API interacts with the connection object.

* Accommodate multiple data sources or messaging queue implementations
	* A distributed transaction manager becomes mandatory. In a JTA environment, the transaction manager must be aware of all logical connections the client has acquired as it has to commit or roll them back according to the global transaction outcome. 
	* By providing logical connections, the application server can decorate the database connection handles with JTA transaction semantics.

* DataSource without connection pooling
    1. The application data layer asks the DataSource for a database connection
    2. The DataSource will use the underlying driver to open a physical connection
    3. A physical connection is created, and a TCP socket is opened
    4. The DataSource under test does not wrap the physical connection, and it simply lends it to the application layer
    5. The application executes statements using the acquired database connection
    6. When the connection is no longer needed, the application closes the physical connection along with the underlying TCP socket.
    
* Connection pooling (Advantages of using database connections)
    * It avoids both the database and the driver overhead for establishing a TCP connection
    * It prevents destroying the temporary memory buffers associated with each database connection
    * It reduces client-side JVM object garbage
    * Prevent Oracle XE connection handling limitation issue
        * The Oracle 11g Express Edition throws the following exception when running very short transactions without using a connection pooling solution:
        * ORA-12516, TNS:listener could not find available handler with matching protocol stack

##### 3.2.1 Why is pooling so much faster?
* The connection pooling mechanism
    1. When a connection is being requested, the pool looks for unallocated connections
    2. If the pool finds a free one, it handles it to the client
    3. If there is no free connection, the pool tries to grow to its maximum allowed size
    4. If the pool already reached its maximum size, it will retry several times before giving up with a connection acquisition failure exception
    5. When the client closes the logical connection, the connection is released and returns to the pool without closing the underlying physical connection.
    
    * The connection pool does not return the physical connection to the client, but instead it offers a proxy or a handle. 
    * When a connection is in use, the pool changes its state to `allocated` to prevent two concurrent threads from using the same database connection. 
    * The proxy intercepts the connection close method call, and it notifies the pool to change the connection state to `unallocated`.
    * The connection pool acts as a bounded buffer for the incoming connection requests. If there is a traffic spike, the connection pool will level it, instead of saturating all the available database resources.

* Configuring the right pool size is important
    * Provisioning the connection pool requires understanding the application-specific database access patterns and also connection usage monitoring.
    
* Two options to avoid system overloading:
    * Discarding the overflowing traffic (affecting availability)
    * Queuing requests and wait for busy resources to become available (increasing response time)
        * The queue is prevented from growing indefinitely and saturating application server resources by putting an upper bound on the connection request wait time.

#### 3.3 Queuing theory capacity planning
* Little’s Law³ is a general-purpose equation applicable to any queueing system being in a stable state (the arrival rate is not greater than the departure rate).
* Average number of requests (L): 
    * L = λ × W
    * L - average number of requests in the system (including both the requests being serviced and the ones waiting in the queue)
    * λ - long-term average arrival rate
    * W - average time a request spends in a system

* TBD...

#### 3.4 Practical database connection provisioning
* FlexyPool: support for monitoring and failover strategies
    * FlexyPool metrics
        * concurrent connection requests:      How many connections are being requested at once
        * concurrent connections:              How many connections are being used at once
        * maximum pool size:                   If the target DataSource uses adaptive pool sizing, this metric shows how the pool size varies with time
        * connection acquisition time:         The time it takes to acquire a connection from the target DataSource
        * overall connection acquisition time: The total connection acquisition interval (including retries) retry attempts The connection acquisition retry attempts
        * overflow pool size:                  How much the pool size can grow over the maximum size until timing out the connection acquisition request
        * connection lease time:               The duration between the moment a connection is acquired and the time it gets released
        
    * Metrics are important for visualizing connection usage trends, in case of an unforeseen traffic spike.
    * The connection acquisition time could reach the DataSource timeout threshold.
    
* Failover strategies
    * The failover mechanism applies various strategies to prevent timed-out connection requests from being discarded.
    * Default failover strategies of FlexyPool:
        * Increment pool size on timeout: 
            * The connection pool has a minimum size and, on demand, it can grow up to its maximum size.
            * This strategy will increment the target connection pool maximum size on connection acquisition timeout.
            * The overflow is a buffer of extra connections allowing the pool to grow beyond its initial maximum size, until it reaches the overflow size threshold.
        * Retrying attempts:
            * This strategy is useful for those connection pools lacking a connection acquiring retry mechanism, and it simply reattempts to fetch a connection for a given number of tries.
##### 3.4.1 A real-life connection pool monitoring example
* How FlexyPool failover strategies can determine the right connection pool size. Use a batch processor as example:
* TBD...


### 4.Batch Updates
### 5.Statement Caching
#### 5.1 Statement lifecycle
* The main database modules responsible for processing an SQL statement are the Parser, the Optimizer and the Executor.

##### Parser
* The statements are verified by Parser both 
	* syntactically (the statement keywords must be properly spelled and following the SQL language guidelines) 
	* and semantically (the referenced tables and column do exist in the database).
	
##### Optimizer
* For a given syntax tree, the database must decide the most efficient data fetching algorithm. Data is retrieved by following an access path.
* Optimizer needs to evaluate multiple data traversing options like:
	* the access method for each referencing table (table scan or index scan)
	* for index scans, it must decide which index is better suited for fetching this result set
	* for each joining relation (e.g. table, views or Common Table Expression), it must choose the best performing join type (e.g. Nested Loops Joins, Hash Joins, Sort Merge Joins)
	* the joining order becomes very important especially for Nested Loops Joins.

* The more time is spent on finding the best possible execution plan, the higher the transaction response time will get, so the Optimizer has a fixed time budget for finding a reasonable plan
* The most common decision-making algorithm is CBO (Cost-Based Optimizer). 
	* Each access method translates to a physical database operation, and its associated cost in resources can be estimated.
	* The database stores various statistics like table sizes and data cardinality (how much the columnvalues differ from one row to the other) to evaluate the cost of a given database operation. 
	* Time is the most common unit of cost, and the database estimates it based on the number of CPU cycles and I/O operations required by a particular execution.
* When finding an optimal execution plan, the Optimizer might evaluate multiple options, and, based on their overall cost, it will choose the one requiring the least amount of time to execute.

###### Execution plan visualization
```
SQL> EXPLAIN PLAN FOR SELECT COUNT(*) FROM post;
SQL> SELECT plan_table_output FROM table(dbms_xplan.display());
```

##### Executor
* The Executor makes use of:
	* the Storage Engine (for loading data according to the current execution plan) 
	* and the Transaction Engine (to enforce the current transaction data integrity guarantees).

* The factors which impact overall transaction performance:
	* In-memory buffer: It allows the database to reduce the I/O contention, therefore reducing transaction response time. 
	* The consistency model: Since locks may be acquired to ensure data integrity, and the more locking, the less the chance for parallel execution.

#### 5.2 Caching performance gain
* The net effect of reusing statements on the overall application performance.
* Statement caching plays a very important role in optimizing high-performance OLTP (Online transaction processing) systems.

#### 5.3 Server-side statement caching
* Because statement parsing and the execution plan generation are resource intensive operations, some database providers offer an execution plan cache. 
	* The statement string value is used as input to a hashing function, and the resulting value becomes the execution plan cache entry key. 
	* If the statement string value changes from one execution to the other, the database cannot reuse an already generated execution plan. 
		* For this purpose, dynamic-generated JDBC Statement(s) are not suitable for reusing execution plans.

#### Forced Parameterization
* Some database systems offer the possibility of intercepting SQL statements at runtime, so that all value literals are replaced with bind variables. 
* This way, the newly parametrized statement can reuse an already cached execution plan.
* To enable this feature, each database system offers a vendor-specific syntax.
	* Oracle: ALTER SESSION SET cursor_sharing=force;
* Server-side prepared statements allow the data access logic to reuse the same execution plan for multiple executions. 
* A PreparedStatement is always associated with a single SQL statement, and bind parameters are used to vary the runtime execution context. 
	* Because PreparedStatement(s) take the SQL query at creation time, the database can precompile the associated SQL statement prior to executing it.

* Precompilation phase
	* During the precompilation phase, the database validates the SQL statement and parses it into a syntax tree. 
	* When it comes to executing the PreparedStatement, the driver sends the actual parameter values, and the database can jump to compiling and running the actual execution plan
* TBD...	
	
#### 5.4 Client-side statement caching

* JDBC driver can reuse already constructed statement objects. The main goals of the client-side statement caching can be summarized as follows:
	* reducing client-side statement processing, which, in turn, lowers transaction response time
	* sparing application resources by recycling statement objects along with their associated database-specific metadata.

* In high-performance OLTP applications, transactions tend to be very short, so even a minor response time reduction can make a difference on the overall transaction throughput.

##### Oracle statement caching
* Oracle JDBC driver supports caching only for PreparedStatement(s) and CallableStatement(s). (SQL String becomes the cache entry key)
* When enabling caching (it is disabled by default), the driver returns a logical statement, so when the client closes it, the logical statement goes back to the cache.

* Implicit and explicit statement caching mechanisms:
	* Both caching options share the same driver storage, which needs to be configured according to the current application requirements.
	* Oracle implicit statement caching
		* The implicit cache can only store statement metadata(which doesn’t change from one execution to the other). 
		* It’s convenient to configure it at the DataSource level (all connections inheriting the same caching properties).

    * Oracle explicit statement caching
        * When using the explicit cache, the data access controls which statements are cacheable.

		
### 6.ResultSet Fetching
### 7.Transactions
#### 7.5 Read-only transactions
* The JDBC Connection defines the setReadOnly(boolean readOnly)⁷ method which can be used to hint the driver to apply some database optimizations for the upcoming read-only transactions. 
* This method shouldn’t be called in the middle of a transaction because the database system cannot turn a read-write transaction into a read-only one (a transaction must start as read-only from the very beginning)
* Oracle
	* According to the JDBC driver documentation the database server does not support read-only transaction optimizations. 
	* Even when the read-only Connection status is set to true, modifying statements are still permitted, and the only way to restrict such statements is to execute the following SQL command:

	```
	connection.setAutoCommit(false);
	try(CallableStatement statement = connection.prepareCall(
	    "BEGIN SET TRANSACTION READ ONLY; END;")) {
	    statement.execute();
	}
	```
	* The SET TRANSACTION READ ONLY command must run after disabling the auto-commit status as otherwise it will only be applied for this particular statement only.

##### Read-only transaction routing
* Setting up a database replication environment is useful for both high-availability (a Slave can replace a crashing Master) and traffic splitting. 
* In a Master-Slave replication topology, the Master node accepts both read-write and read-only transactions, while Slave nodes only take read-only traffic.

#### 7.6 Transaction boundaries
* When the unit of work consists of multiple SQL statements, the database should wrap them all in a single unit of work.
* By default, every Connection starts in auto-commit mode, each statement being executed in a separate transaction. Unfortunately, it doesn’t work for multi-statement transactions as it moves atomicity boundaries from the logical unit of work to each individual statement.
* Auto-commit should be avoided as much as possible, and, even for single statement transactions, it’s good practice to mark the transaction boundaries explicitly.



## III.JPA and Hibernate
### 8.Why JPA and Hibernate matter
### 9.Connection Management and Monitoring
### 10.Mapping Types and Identifiers
### 11.Relationships
### 12.Inheritance