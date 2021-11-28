# Transaction Management: Concurrency Control

## Transactions
* Ensures ACID
    * `Concurrency manager` for C and I
    * `Recovery manager` for A and D

* Transaction Lifecycle
    1. Begin
    2. End statement
        – If spanning across multiple statements
    3. Commit or rollback

* Lifecycle Listeners
    * Tx lifecycle listener – Takes actions to tx life cycle events
    * Buffer manager
        * On tx rollback/commit: unpins all pages pinned by the current tx
        * Registered itself as a life cycle listener on start of each tx
    
    * Recovery manager
        * Commit: flushes dirty pages and then commit log
        * Rollback: undo all modifications by reading log records
        
 ## Concurrency Manager - Ensures `consistency` and `isolation`
 * `Consistency`
    * Txs will leave the database in a consistent state
    * I.e., all integrity constraints (ICs) are meet
        * Primary and foreign key constrains
        * Non-null constrain
        * (Field) type constrain
        * …
    * Users are responsible for issuing “valid” txs
    
 * `Isolation`
 	* Interleaved execution of txs should have the net effect identical to executing tx in some serial order.
 	* T1 and T2 are executed concurrently, isolation gives that the net effect to be equivalent to either T1 followed by T2 or T2 followed by T1.
 	* The DBMS does not guarantee the result in which particular order.

* Concurrent Txs
    * Pros:
        * Increases throughput (via CPU and I/O pipelining)
        * Shortens response time for short txs
    * But operations must be interleaved correctly

## Transactions and Schedules
* A schedule is a list of actions/operations from a set of transactions.
* Serial schedule: If the actions of different transactions are not interleaved, we call this schedule a serial schedule.
* Equivalent schedules: The effect of executing the first schedule is identical to the effect of executing the second schedule.
* Serializable schedule: A schedule that is equivalent to some serial execution of the transactions.
* Goal: Interleave operations while making sure the schedules are serializable.

## Anomalies
* Weird situations that would happen when interleaving operations. But not in serial schedules.
* Mainly due to the conflicting operations.
### Conflicting operation
* Two operations on the same object are conflict if they are operated by different txs and at least one of these operations is a write.
* Types: 
	* Write-read conflict: Dirty reads, unrecoverable schedule
	* Read-write conflict: Unrepeatable reads
	* Write-write conflict: lost updates
### Avoiding Anomalies
* To ensure conflict serializability and recoverable schedule.
* Recoverable schedule: A schedule is recoverable if each tx T commits only after all txs whose changes T reads, commit.
	* Avoid cascading aborts
	* Disallow a tx from reading uncommitted changes from other txs.

## Lock-based concurrency control
* For isolation and consistency, a DBMS should only allow serializable, recoverable schedules.
	* Uncommitted changes cannot be seen(no WR)
	* Ensure repeatable read(no RW)
	* Cannot overwrite uncommitted change(no WW)
* What type of lock to get for each operation?
* When should a transaction acquire/release lock?

### 2PL and S2PL
* Types of lock
	* Shared(S) lock
	* Exclusive(X) lock
#### Two Phase Lock Protocol(2PL)
* Phase 1: Growing Phase
	* Each tx must obtain an S(X) lock on an object before reading/writing it.
* Phase 2: Shrinking Phase
	* A transaction can not request additional locks once it releases any locks.
* Ensures conflict serializability

##### Implementation
* Lock and unlock requests are handled by the lock manager. Which is shared between concurrency managers.
* Lock table entry
	* Number of transactions currently holding a lock.
	* Type of lock held
	* Pointer to queue of lock requests
* Locking and unlocking have to be atomic operations.

* Sample Implementation of Lock Table
	* Implemented as an in-memory hash table indexed on the name of the data item being locked.
	* New lock request is added to the end of the queue of requests for the data item.
	* Request is granted if it is compatible with all earlier requests.

* Problems of 2PL	
	* Cascading rollback
	* Deadlock

#### Strict Two-Phase Locking(S2PL)
* Each tx obtains locks as in the growing phase in 2PL.
* But the tx `holds all locks until it completes`.(Only growing phase, shrinking phase happens when the transaction is ending)
* Allows only serializable and strict schedules.
* Strict schedules
	* A schedule is strict if for any two txs T1 and T2, if a write operation of T1 precedes a conflicting operation of T2(either read or write), then Y1 commits before that conflicting operation of T2.
	* Avoids cascading rollback, but still has deadlock.

### Deadlock
* Detect `Waits-for` graph when acquiring locks(or buffers)
* Other techniques
	* Timeout & rollback(deadlock detection)
		* Assume Ti wants a lock that Tj holds
		* Ti waits for the lock
		* If Ti stays on the wait list too long thenL Ti is rolled back
	* Wait-die(deadlock prevention)
		* Assume each Ti has a timestamp(e.g. tx number)
		* If Ti want a lock that Tj holds
		* If Ti is older than Tj, it waits for Tj
		* Otherwise Ti aborts
	* Conservative locking(deadlock prevention)
		* Every Ti locks all objects at once(atomically) in the beginning
		* No interleaving for conflicting txs--performs well only if there is no/very few long txs(e.g. in-memory DBMS)
		* How to know which objects to lock before tx execution?
			* Requires the coder of a stored procedure to specify its read- and write-sets explicitly.
			* Does not support ad-hoc queries.


### Granularity of locks
* Granularity of locking objects – Records vs. blocks vs. tables/files
    * Fine granularity: high concurrency, high locking overhead
    * Coarse granularity: low locking overhead, low concurrency

* Reducing Locking Overhead
    * Data “containers” are nested: Database -> tables -> pages -> tuples
    * e.g., When scanning, can we lock a file instead of all contained blocks/records to reduce the locking overhead? -> MGL(Multiple-Granularity Locks)
   
#### Multiple-Granularity Locks
* Multiple-granularity locking (MGL) allows users to set locks on objects that contain other objects
    * Locking a file implies locking all contained blocks/records
    * How does a lock manager know if a file is lockable?
        * Some other tx may hold a conflicting lock on a block in that file

* Checking If An Object Is Locked

    * Solution 1:To lock a file, check whether all blocks/records in that file are locked
        * Bad strategy - Does not save the locking overhead

    * Allow transactions to lock at each level, but `with a special protocol using new “intention” locks`:
        * `Intention-shared` (IS)
            * Indicates explicit locking at a lower level of the tree but only with shared locks
        * `Intention-exclusive` (IX)
            * Indicates explicit locking at a lower level with exclusive or shared locks
        * `Shared and intention-exclusive` (SIX)
            * The subtree rooted by that node is locked explicitly in shared mode and explicit locking is being done at
              a lower level with exclusive-mode locks

* Locks are acquired in `root-to-leaf` order
* Locks need to be released in `leaf-to-root` order

## Dynamic Databases

* The database can grow and shrink through the `insertions` and `deletions`.
    * Any trouble? `Phantoms`

### Phantom

* Phantoms Caused by Insertion
    ```
    – T1: SELECT * FROM users WHERE age=10;
    – T2: INSERT INTO users VALUES (3, 'Bob', 10); COMMIT;
    – T1: SELECT * FROM users WHERE age=10;
    ```
    * A transaction that reads the entire contents of a table multiple times will see different data.
        * e.g., in a join query

* Phantoms Caused by Update
    ```
    – T1: SELECT * FROM users WHERE age=10;
    – T2: UPDATE users SET age=10 WHERE id=7;
    COMMIT;
    – T1: SELECT * FROM users WHERE age=10;
    ```
    * T1 only share locks the records with the age equals to 10
    * The record with id=7 is not in the locking item set of T1, so T2 can update this record

#### How to Prevent Phantoms?

* EOF locks or multi-granularity locks
    * X-lock the containing file when inserting/updating records in a block
    * Hurt performance (no concurrent inserts/updates)
    * Usually used to prevent phantoms by insert
    * But `not` phantoms by update

* Index (or predicate) locking
    * Prevent phantoms caused by both insert and update
    * Works only if indices for the inserting/updating fields are created

### Isolation levels

* Transaction Characteristics
    * SQL allows users to specify the followings:
        * `Access model`
            * READ ONLY or READ WRITE
            * By Connection.setReadOnly() in JDBC
        * `Isolation level`
            * Trade anomalies for better tx concurrency
            * By Connection.setTransactionIsolation()

* Defined by the ANSI/ISO SQL standard:

| Isolation level  | Dirty reads | Unrepeatable reads | Phantoms |
| ---------------- | ----------- | ------------------ | -------- | 
| Read Uncommitted | Maybe       | Maybe              | Maybe    |
| Read Committed   | No          | Maybe              | Maybe    |
| Repeatable Read  | No          | No                 | Maybe    |
| Serializable     | No          | No                 | No       |

* How to implement these using a locking protocol?

| Isolation level  | Shared Lock        | Predicate Lock     |
| ---------------- | ------------------ | ------------------ |  
| Read Uncommitted | No                 | No                 | 
| Read Committed   | Released early     | No                 |
| Repeatable Read  | Held to completion | No                 |
| Serializable     | Held to completion | Held to completion |

## Meta-structures

* DBMS maintains some meta-structures in addition to data perceived by users
    * e.g., FileHeaderPage in RecordFile

* Concurrency Control of Access to Meta-Structures
    * Access to FileHeaderPage?
        * Whenever insertions/deletions of records happen
    * How to lock FileHeaderPage?
        * S2PL?
    * S2PL will serialize all insertions and deletions
        * Hurts performance if we have many inserts/deletes

* Early Lock Release
    * Actually, lock of FileHeaderPage can be `released early`
        * No “data” revealed; no hurt to I
    * Locking steps for a (logical) insertion/deletion:
        * Acquire locks of FileHeaderPage and target object (RecordPage or a record) in order
            * Perform changes
                * `Release` the lock of FileHeaderPage (but not the object)
    * Better concurrency for I
    * No harm to C
    * Needs special care to ensure A and D

# References

* [Introduction to Database System by Shan-Hung Wu](https://www.youtube.com/playlist?list=PLS0SUwlYe8cyln89Srqmmlw42CiCBT6Zn)
