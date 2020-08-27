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
* Multiple DML statements can be grouped into a single database request.
    * Sending multiple statements in a single request reduces the number of database roundtrips, therefore decreasing transaction response time.
#### 4.1 Batching Statements

#### 4.2 Batching PreparedStatements
* Because a PreparedStatement is associated with a single DML statement, the batch update can group multiple parameter values belonging to the same prepared statement.

##### 4.2.1 Choosing the right batch size
* Finding the right batch size is not a trivial thing to do as there is no mathematical equation to solve the appropriate batch size for any enterprise application.
* Measuring the application performance gain in response to a certain batch size value remains the most reliable tuning option.
* Even a low batch size can reduce the transaction response time, and the performance gain doesn’t grow linearly with batch size. 
* Although a larger batch value can save more database roundtrips, the overall performance gain remains relatively flat and can even drop for larger batch sizes.
* You should always measure the performance improvement for various batch sizes. In practice, a relatively low value (between 10 and 30) is usually a good choice.

##### 4.2.2 Bulk operations
* SQL offers bulk operations to modify all rows that satisfy a given filtering criteria. 
* Bulk update or delete statements can also benefit from indexing, just like select statements.
* Bulk v.s Batch
	* The bulk alternative is one order of magnitude faster than batching.
	* Batch updates are more flexible since each row can take a different update logic. 
	* Batch updates can also prevent lost updates if the data access logic employs an optimistic locking mechanism.
	* Like with updates, bulk deleting is also much faster than deleting in batches.

* Bulk processing caveats
	* Processing too much data in a single transaction can degrade application performance, especially in a highly concurrent environment. 
	* Writes always block other conflicting writes. (if using locks (two-phase locking) or MVCC (Multiversion Concurrency Control)) 
	* In case the bulk updated records conflict with other concurrent transactions, then either the bulk update transaction might have to wait for some row-level locks to be released or other transactions might wait for the bulk updated rows to be committed.
	
#### 4.3 Retrieving auto-generated keys
* It’s important to know that auto-generated database identifiers might conflict with the batch insert process
* Two auto incremented identifier strategies: an identity column or a database sequence generator
* Not all database systems support fetching auto-generated keys from a batch of statements.

##### 4.3.1 Sequences to the rescue
* Database sequences offer the advantage of decoupling the identifier generation from the actual row insert. 
* To make use of batch inserts, the identifier must be fetched prior to setting the insert statement parameter values.
```
Oracle
SELECT post_seq.NEXTVAL FROM dual;
```
* Because the primary key is generated up-front, so batch inserts are not driver dependent anymore.
* Sequence number generation optimizations
	* It lowers the sequence call execution as much as possible. 
	* If the number of inserted records is relatively low, then the sequence call overhead (extra database roundtrips) is insignificant.

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
* the following JPA attributes have a peculiar behavior, which can surprise someone who’s familiar with the JPA specification only:
	* the FlushModeType.AUTO¹ doesn’t trigger a flush for native SQL queries, like it does for JPQL or Criteria API
	* the FetchType.EAGER² might choose an SQL join or a secondary select, whether the entity is fetched directly from the EntityManager or through a JPQL (Java Persistence Query Language) or a Criteria API query.

### 8.3 Schema ownership
* Although it might not fit any enterprise system, having the database as a central integration point can still be a choice for many reasonable size enterprise systems.
* The relational database concurrency models offer strong consistency guarantees, therefore having a significant advantage to application development. 
	* If the integration point doesn’t provide transactional semantics, it’s much more difficult to implement a distributed concurrency control mechanism.

#### The distributed commit log
* For very large enterprise systems, where data is split among different providers (relational database systems, caches, Hadoop, Spark), it’s no longer possible to rely on the relational database to integrate all disparate subsystems.

* In this case, `Apache Kafka` offers a fault-tolerant and scalable append-only log structure, which every participating subsystem can read and write concurrently.
	* The commit log becomes the integration point, each distributed node individually traversing it and maintaining client-specific pointers in the sequential log structure.
	* This design resembles a database replication mechanism, and so it offers 
		* durability (the log is persisted on disk)
		* write performance (append-only logs don’t require random access) and 
		* read performance (concurrent reads don’t require blocking) as well.

* No matter what architecture style is chosen, there is still a need to correlate the transient Domain Model with the underlying persistent data.

### 8.4 Write-based optimizations
* JPA entity states
	* New (Transient), Managed (Persistent), Detached, Removed.

* The Persistence Context captures entity state changes, and, during flushing, it translates them to SQL statements.
* The JPA EntityManager⁴ and the Hibernate Session⁵ (which includes additional methods for moving an entity from one state to the other) interfaces are gateways towards the underlying Persistence Context, and they define all the entity state transition operations.

#### SQL injection prevention
* Hibernate uses PreparedStatement(s) exclusively, so not only it protect against SQL injection, but the data access layer can better take advantage of server-side and client-side statement caching as well.
#### Write-behind cache
* The Persistence Context acts as a transactional write-behind cache, deferring entity state flushing up until the last possible moment
* Because every modifying DML statement requires locking (to prevent dirty writes), the write behind cache can reduce the database lock acquisition interval, therefore increasing concurrency.
* But caches introduce consistency challenges, and the Persistence Context requires a flush prior to executing any JPQL or native SQL query (as otherwise it might break the read-your-own-write consistency guarantee).
* Hibernate doesn’t automatically flush pending changes when a native query is about to be executed, and the application developer must explicitly instruct what database tables are needed to be synchronized.

#### Transparent statement batching
* Since all changes are being flushed at once, Hibernate may benefit from batching JDBC statements.
* Batch updates can be enabled transparently, even after the data access logic has been implemented.
* With just one configuration, Hibernate can execute all prepared statements in batches.

#### Application-level concurrency control
* Optimistic locking
	* The JPA optimistic locking mechanism allows preventing lost updates because it imposes a happens before event ordering. 
	* But in multi-request conversations, optimistic locking requires maintaining old entity snapshots, and JPA makes it possible through Extended Persistence Contexts or detached entities.
	* Persistence Context
		* A Java EE application server can preserve a given Persistence Context across several web requests, therefore providing application-level repeatable reads.
		* But this strategy is not free since the application developer must make sure the Persistence Context is not bloated with too many entities, which, apart from consuming memory, it can also affect the performance of the Hibernate default dirty checking mechanism.
		
* Merging detached entities
	* JPA allows merging detached entities, which rebecome managed and automatically synchronized with the underlying database system.
	
* Pessimistic locking
	* JPA also supports a pessimistic locking query abstraction, which comes in handy when using lowerlevel transaction isolation modes.
	* Hibernate has a native pessimistic locking API, which brings support for timing out lock acquisition requests or skipping already acquired locks.

### 8.5 Read-based optimizations
* Following the SQL standard, the JDBC ResultSet is a tabular representation of the underlying fetched data. The Domain Model being constructed as an entity graph, the data access layer must transform the flat ResultSet into a hierarchical structure.
* The source of data is not an in-memory repository, and the fetching behavior influences the overall data access efficiency.

#### The fetching responsibility
* Each business use case has different data access requirements, and one policy cannot anticipate all possible use cases, so the fetching strategy should always be set up on a query basis.

#### Prefer projections for read-only views
* Sometimes a custom projection (selecting only a few columns from an entity) is much more suitable, and the data access logic can even take advantage of database specific SQL constructs that might not be supported by the JPA query abstraction.

#### The second-level cache
* First-level cache
	* If the Persistence Context acts as a transactional write-behind cache, its lifetime is bound to that of a logical transaction. 
	* For this reason, the Persistence Context is also known as the first-level cache, and so it cannot be shared by multiple concurrent transactions.
* Second-level cache
	* The second-level cache is associated with an EntityManagerFactory, and all Persistence Contexts have access to it.
	* The second-level cache can store entities as well as entity associations (one-to-many and many-to-many relationships) and even entity query results.
* Each provider takes a different approach to caching (as opposed to EclipseLink, by default, Hibernate disables the second-level cache).
* Although the second-level cache can mitigate the entity fetching performance issues, it requires a distributed caching implementation, which might not elude the networking penalties anyway.



### 9.Connection Management and Monitoring

#### 9.1 JPA connection management
* The Java EE DataSource can be located through JNDI. In the persistence.xml configuration file.
* The application developer must supply the JNDI name of the associated JTA or RESOURCE_LOCAL DataSource. 
* The transaction-type attribute must also match the data source transaction capabilities.

* For a Java EE application it’s perfectly fine to rely on the application server for providing a fullfeatured DataSource reference.
* Stand-alone applications are usually configured using dependency injection rather than JNDI.
* Because most enterprise applications need connection pooling and monitoring capabilities anyway. 
* For this reason, JPA connection management is still an implementation specific topic, and the upcoming sections will dive into the connection provider mechanism employed by Hibernate.

##### 9.3.1 Hibernate statistics
* Hibernate statistics gathers notifications related to database connections, Session transactions and even second-level caching usage.
* For performance reasons, the statistics mechanism is disabled by default.
* To enable the statistics gathering mechanism and print the info into the current application log:
```
<property name="hibernate.generate_statistics" value="true"/>
<logger name="org.hibernate.engine.internal.StatisticalLoggingSessionEventListener" level="info" />
```
* Whenever a Hibernate Session (Persistence Context) ends, the report will be displayed in the current running log.
* The application developer can supply its own custom StatisticsImplementor implementation.

* Metrics framework
	* In a high-throughput transaction system, the amount of metric data needed to be recorded can be overwhelming, so storing all these values into memory is not really practical at all.
	* To reduce the memory footprint, the metrics framework might uses various reservoir sampling strategies, that either employ a fixed-size sampler or a time-based sampling window.
	* The framework supports a great variety of metrics (e.g. timers, histograms, gauges).
	* The framework can use multiple reporting channels as well (e.g. SLF4J, JMX, Ganglia, Graphite).
	* For all these reasons, it’s better to use a mature framework such as Dropwizard Metrics instead of building a custom implementation from scratch.

##### 9.3.1.1 Customizing statistics
* TBD...

#### 9.4 Statement logging
* It is the developer responsibility to validate the effectiveness and overall performance impact of the DML statements automatically generated by the ORM tool.
* SQL statement logging becomes relevant from the early stages of application development.
* When a business logic is implemented, the Definition of Done should include a review of all the associated data access layer operations.
* Hibernate logs all SQL statements on a debug level in the org.hibernate.SQL logging hierarchy. 
```
<logger name="org.hibernate.SQL" level="debug"/>
```
##### 9.4.1 Statement formatting
##### 9.4.2 Statement-level comments

### 10.Mapping Types and Identifiers
* JPA uses three main Object-Relational mapping elements: type, embeddable and entity.
* Entity v.s. Embeddable
* Both the entity and the embeddable group multiple Domain Model properties by relying on Hibernate Type(s) to associate database column types with Java value objects (e.g. String, Integer, Date). 
* Identifiers are mandatory for entity elements, and an embeddable type is forbidden to have an
identity of its own. 
* The composition association, defined by UML, is the perfect analogy for the relationship between an entity and an embeddable. 
* Lacking an identifier, the embeddable object cannot be managed by a Persistence Context, and its state is controlled by its parent entity.

#### 10.2 Identifiers
* Natural key: The primary key can have a meaning in the real world.
	The natural key can be composed of one or multiple columns. 
		* Compound natural keys might incur an additional performance penalty because multi-column joins are slower than single-column ones.
		* Multi-column indexes have a bigger memory footprint too.
* Surrogate identifier: The primary key generated synthetically.

### 11.Relationships

##### Mapping collections
* Although @OneToMany, @ManyToMany or @ElementCollection are convenient from a data access perspective (entity state transitions can be cascaded from parent entities to children), they are definitely not free of cost. 
* The price for reducing data access operations is paid in terms of result set fetching flexibility and performance. 
A JPA collection binds a parent entity to a query that usually fetches all the associated child records.
* Because of this, the entity mapping becomes sensitive to the number of child entries.

* Even if a collection is fetched lazily, Hibernate might still require to fully load each entity when the collection is accessed for the first time. 
* Although Hibernate supports extra lazy collection fetching, this is only a workaround and doesn’t address the root problem.

* Alternatively, every collection mapping can be replaced by a data access query, which can use an SQL projection that’s tailored by the data requirements of each business use case. This way, the query can take business case specific filtering criteria.

### 12.Inheritance