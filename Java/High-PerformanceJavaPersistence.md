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

### 4.Batch Updates
### 5.Statement Caching
### 6.ResultSet Fetching
### 7.Transactions

## III.JPA and Hibernate
### 8.Why JPA and Hibernate matter
### 9.Connection Management and Monitoring
### 10.Mapping Types and Identifiers
### 11.Relationships
### 12.Inheritance