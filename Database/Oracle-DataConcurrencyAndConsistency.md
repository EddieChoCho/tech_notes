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

### Overview of Locking Mechanisms

* Locks are mechanisms that prevent destructive interaction between transactions accessing the same resource.
* Resources include two general types of objects:
  * User objects, such as tables and rows (structures and data)
  * System objects not visible to users, such as shared data structures in the memory and data dictionary rows

## How Oracle Manages Data Concurrency and Consistency

### Multiversion Concurrency Control

###  

## How Oracle Locks Data

## Overview of Oracle Flashback Query

## References

* [11 Data Concurrency and Consistency](https://docs.oracle.com/en/database/oracle/oracle-database/21/cncpt/data-concurrency-and-consistency.html)
