# Oracle - Data Concurrency and Consistency

## Introduction to Data Concurrency and Consistency

* A multiuser database must provide the following:
  * The assurance that users can access data at the same
    time ([data concurrency](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/glossary.html#GUID-D7E696DB-944C-4798-B70D-5C2381FE971F))
  * The assurance that each user sees a consistent view of the
    data ([data consistency](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/glossary.html#GUID-B016467E-5663-4AC8-B54D-181CA1B8198E))
    , including visible changes made by the user's own transactions and committed transactions of other users

* Real-world considerations usually require a compromise between perfect transaction isolation and performance.
* Oracle Database maintains data consistency by using a multiversion consistency model and various types of locks and
  transactions.
  * The database can present a view of data to multiple concurrent users, with each view consistent to a point in time.
  * Because different versions of data blocks can exist simultaneously, transactions can read the version of data
    committed at the point in time required by a query and return results that are consistent to a single point in time.

### Multiversion Read Consistency

* Queries of an Oracle database have the following characteristics:
  * Read-consistent queries: The data returned by a query is committed and consistent for a single point in time.
  * Nonblocking queries: Readers and writers of data do not block one another.

#### Statement-Level Read Consistency

* Oracle Database always enforces statement-level read consistency, which guarantees that data returned by a single
  query is committed and consistent for a single point in time.
* The point in time to which a single SQL statement is consistent depends on the transaction isolation level and the
  nature of the query:
  * Read committed
    * This point is the time at which the `statement was opened`.
    * For example, if a SELECT statement opens at SCN 1000, then this statement is consistent to SCN 1000.
  * Serializable or Read-only transaction
    * This point is the time the transaction began.
    * For example, if a transaction begins at SCN 1000, and if multiple SELECT statements occur in this transaction,
      then each statement is consistent to SCN 1000.

  * In a Flashback Query operation (SELECT ... AS OF)
    * The SELECT statement explicitly specifies the point in time.
    * For example, you can query a table as it appeared last Thursday at 2 p.m.

#### Transaction-Level Read Consistency

* Oracle Database can also provide read consistency to all queries in a transaction, known as transaction-level read
  consistency.
* In this case, each statement in a transaction sees data from the same point in time. This is the time at which the
  transaction began.
* Transaction-level read consistency produces repeatable reads and does not expose a query to phantom reads.

#### Read Consistency and Undo Segments

* To manage the multiversion read consistency model, the database must create a read-consistent set of data when a table
  is simultaneously queried and updated.
* Oracle Database achieves read consistency through undo data.
  * Undo data
    * Records of the actions of transactions, primarily before they are committed.
    * The database can use undo data to logically reverse the effect of SQL statements.
    * Undo data is stored in undo segments.
* Whenever a user modifies data, Oracle Database creates undo entries, which it writes to undo segments.
* The undo segments contain the old values of data that have been changed by uncommitted or recently committed
  transactions. Thus, multiple versions of the same data, all at different points in time, can exist in the database.
* The database can use snapshots of data at different points in time to provide read-consistent views of the data and
  enable nonblocking queries.

* Oracle Real Application Clusters (Oracle RAC)
  * Read consistency is also guaranteed in Oracle RAC environments.
  * Oracle RAC uses a cache-to-cache block transfer mechanism known as cache fusion to transfer read-consistent images
    of data blocks from one database instance to another.

##### Read Consistency: Example

* TBD

##### Read Consistency and Interested Transaction Lists

* TBD

### Preventable Phenomena and Transaction Isolation Levels

* The three preventable phenomena are:
  * Dirty reads: A transaction reads data that has been written by another transaction that has not been committed yet.
  * Nonrepeatable (fuzzy) reads: A transaction rereads data it has previously read and finds that another committed
    transaction has modified or deleted the data.
  * Phantom reads (or phantoms): A transaction re-runs a query returning a set of rows that satisfies a search condition
    and finds that another committed transaction has inserted additional rows that satisfy the condition.

| Isolation Level  | Dirty Read      | Nonrepeatable Read | Phantom Read |
| ---------------- | ------------ | ------------------ | ------------ |  
| Read uncommitted | Possible     | Possible           | Possible     |
| Read committed   | Not possible | Possible           | Possible     |
| Repeatable read  | Not possible | Not possible       | Possible     |
| Serializable     | Not possible | Not possible       | Not possible |

### Overview of the Oracle Database Locking Mechanism

* Locks are mechanisms that prevent destructive interaction between transactions accessing the same resource.
* Resources include two general types of objects:
  * User objects, such as tables and rows (structures and data)
  * System objects not visible to users, such as shared data structures in the memory and data dictionary rows

#### Summary of Locking Behavior

* In general, the database uses two types of locks: exclusive locks and share locks.
* Locks affect the interaction of readers and writers.
  * A reader is a query of a resource.
  * A writer is a statement modifying a resource.
* The following rules summarize the locking behavior of Oracle Database for readers and writers:
  * A row is locked only when modified by a writer.
    * Note: Under normal circumstances, the database does not escalate a row lock to the block or table level.
  * A writer of a row blocks a concurrent writer of the same row.
  * A reader never blocks a writer.
    * Note: The only exception is a SELECT ... FOR UPDATE statement, which is a special type of SELECT statement that
      does lock the row that it is reading.
    * Note:Readers of data may have to wait for writers of the same data blocks in very special cases of pending
      distributed transactions.
  * A writer never blocks a reader.
    * When a row is being changed by a writer, the database uses undo data to provide readers with a consistent view of
      the row.

#### Use of Locks

* Locks achieve the following important database requirements:
  * Consistency: The data a session is viewing or changing must not be changed by other sessions until the user is
    finished.
  * Integrity: The data and structures must reflect all changes made to them in the correct sequence.
* Because the locking mechanisms of Oracle Database are tied closely to transaction control, application designers need
  only define transactions properly, and Oracle Database automatically manages locking.
* Users never need to lock any resource explicitly.

#### Lock Modes

* Oracle Database
  automatically `uses the lowest applicable level of restrictiveness to provide the highest degree of data concurrency yet also provide fail-safe data integrity`
  .
* Oracle Database uses two modes of locking in a multiuser database: Exclusive lock mode, Share lock mode
* Assume that a transaction uses a ```SELECT ... FOR UPDATE``` statement to select a single table row. The transaction
  acquires an exclusive row lock and a row share table lock.
  * The row lock allows other sessions to modify any rows other than the locked row.
  * The table lock prevents sessions from altering the structure of the table.
  * Thus, the database permits as many statements as possible to execute.

#### Lock Conversion and Escalation

* Lock Conversion
  * In lock conversion, the database automatically converts a table lock of lower restrictiveness to one of higher
    restrictiveness.
  * `Oracle Database performs lock conversion as necessary.`
    * For example, suppose a transaction issues a ```SELECT ... FOR UPDATE``` for an employee and later updates the
      locked row.
      * In this case, the database automatically converts the row share table lock to a row exclusive table lock.
      * Because row locks are acquired at the highest degree of restrictiveness, no lock conversion is required or
        performed.

* Lock Escalation
  * Lock escalation occurs when numerous locks are held at one level of granularity (for example, rows) and a database
    raises the locks to a higher level of granularity (for example, table).
  * If a session locks many rows in a table, then some databases automatically escalate the row locks to a single table.
  * The number of locks decreases, but the restrictiveness of what is locked increases.
  * `Oracle Database never escalates locks.`
    * Lock escalation greatly increases the probability of deadlocks.
    * Assume that a system is trying to escalate locks on behalf of transaction 1 but cannot because of the locks held
      by transaction 2. A deadlock is created if transaction 2 also requires lock escalation of the same data before it
      can proceed.

#### Lock Duration

* `Oracle Database automatically releases a lock when some event occurs so that the transaction no longer requires the resource`
  .
* Usually, the database holds locks acquired by statements within a transaction for the duration of the transaction.
  * Note:
    * A table lock taken on a child table because of an unindexed foreign key is held for the duration of the statement,
      not the transaction.
    * Also, the DBMS_LOCK package enables user-defined locks to be released and allocated at will and even held over
      transaction boundaries.

#### Locks and Deadlocks

* Oracle Database automatically detects deadlocks and resolves them by rolling back one statement involved in the
  deadlock, releasing one set of the conflicting row locks.
  * The database returns a corresponding message to the transaction that undergoes `statement-level rollback`.
  * The statement rolled back belongs to the transaction that detects the deadlock. Usually, the signaled transaction
    should be rolled back explicitly, but it can retry the rolled-back statement after waiting.

* Deadlocks most often occur when transactions explicitly override the default locking of Oracle Database.
* Because Oracle Database does not escalate locks and does not use read locks for queries, but does use row-level (
  rather than page-level) locking, deadlocks occur infrequently.

### Overview of Automatic Locks

#### DML Locks

* A DML lock, also called a data lock, guarantees the integrity of data accessed concurrently by multiple users.
* DML statements automatically acquire the following types of locks:
  * Row Locks (TX)
    * A transaction acquires a row lock for each row modified by an INSERT, UPDATE, DELETE, MERGE, or SELECT ... FOR
      UPDATE statement.
      * Note: If a transaction terminates because of database instance failure, then block-level recovery makes a row
        available before the entire transaction is recovered.
    * If a transaction obtains a lock for a row(e.g., exclusive lock ), then the transaction also acquires a lock for
      the table containing the row(sub-exclusive lock).
    * Storage of Row Locks
      * Unlike some databases, which use a lock manager to maintain a list of locks in memory, Oracle Database stores
        lock information in the data block that contains the locked row.
      * The database uses a queuing mechanism for acquisition of row locks. If a transaction requires a lock for an
        unlocked row, then the transaction places a lock in the data block.
      * Each row modified by this transaction points to a copy of the transaction ID stored in the block header.
      * When a transaction ends, the transaction ID remains in the block header. If a different transaction wants to
        modify a row, then it uses the transaction ID to determine whether the lock is active.
      * If the lock is active, then the session asks to be notified when the lock is released. Otherwise, the
        transaction acquires the lock.

  * Table Locks (TM)
    * A TM lock is acquired by a transaction when a table is modified by an INSERT, UPDATE, DELETE, MERGE, SELECT with
      the FOR UPDATE clause, or LOCK TABLE statement.
    * DML operations require table locks to reserve DML accesses to the table on behalf of a transaction and to prevent
      DDL operations that would conflict with the transaction.
    * A table lock can be held in any of the following modes: Row Share (RS), Row Exclusive Table Lock (RX), Share Table
      Lock (S), hare Row Exclusive Table Lock (SRX), and Exclusive Table Lock (X).

* Locks and Foreign Keys
  * Locking behavior depends on whether foreign key columns are indexed.
  * If foreign keys are not indexed, then the child table will probably be locked more frequently, deadlocks will occur,
    and concurrency will be decreased.
  * For this reason foreign keys should almost always be indexed.
  * The only exception is when the matching unique or primary key is never updated or deleted.

  * Locks and Unindexed Foreign Keys
    * The database acquires a full table lock on the child table when no index exists on the foreign key column of the
      child table, and a session modifies a primary key in the parent table (for example, deletes a row or modifies
      primary key attributes) or merges rows into the parent table.
    * When both of the following conditions are true, the database acquires a full table lock on the child table:
      * No index exists on the foreign key column of the child table.
      * A session modifies a primary key in the parent table (for example, deletes a row or modifies primary key
        attributes) or merges rows into the parent table.

  * Locks and Indexed Foreign Keys
    * TBD

#### DDL Locks

#### System Locks

## References

* [11 Data Concurrency and Consistency](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-concurrency-and-consistency.html)
