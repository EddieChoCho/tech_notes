# An Empirical Evaluation of In-Memory Multi-Version Concurrency Control

## 1. INTRODUCTION

* Scaling MVCC in a multi-core and in-memory setting is non-trivial: when there are a large number of threads running in
  parallel, the synchronization overhead can outweigh the benefits of multi-versioning.
* Multi-versioning allows read-only transactions to access older versions of tuples without preventing read-write
  transactions from simultaneously generating newer versions. Contrast this with a single-version system where
  transactions always overwrite a tuple with new information whenever they update it.
* Despite all these newer systems using MVCC, there is no one “standard” implementation. There are several design
  choices that have different trade-offs and performance behaviors.
* Key transaction management design decisions in of MVCC DBMSs: (1) concurrency control protocol, (2) version storage, (
  3) garbage collection, and (4) index management.
* Implemented state-of-the-art variants of all of these in an in-memory DBMS and evaluated them using OLTP(Online
  Transaction Processing) workloads. The analysis identifies the fundamental bottlenecks of each design choice.

## 2. BACKGROUND

### 2.1. MVCC Overview

* Advantages of a multi-version system
  1. It can potentially allow for greater concurrency than a single-version system: allows a transaction to read an
     older version of an object at the same time that another transaction updates that same object. This is important in
     that execute read-only queries on the database at the same time that read-write transactions continue to update it.
  2. If the DBMS never removes old versions, then the system can also support “time-travel” operations that allow an
     application to query a consistent snapshot of the database as it existed at some point of time in the past.

* Additional computation and storage overhead.
* note: we only consider serializable transaction execution in this paper.
* logging and recovery are excluded from this study because there is nothing about it that is different from a
  single-version system and in-memory DBMS logging is already covered elsewhere.

### 2.2. DBMS Meta-Data

* Transactions
  * The DBMS assigns a transaction T a unique, monotonically increasing timestamp as its identifier (Tid) when they
    first enter the system.
  * The concurrency control protocols use this identifier to mark the tuple versions that a transaction accesses.
* Tuples
  * Header:
    * txn-id:
      * zero when the tuple is not write-locked.
      * Any transaction that attempts to update A is aborted if this txn-id field is neither zero or not equal to its
        Tid.
    * begin-ts:
      * timestamps that represent the lifetime of the tuple version.
      * The DBMS sets a tuple’s begin-ts to INF when the transaction deletes it.
    * end-ts: timestamps that represent the lifetime of the tuple version
    * pointer: stores the address of the neighboring (previous or next) version (if any).
  * Content:
    * columns

## 3. CONCURRENCY CONTROL PROTOCOL

* This protocol determines:
  1. whether to allow a transaction to access or modify a particular tuple version in the database at runtime.
  2. whether to allow a transaction to commit its modifications.

* We omit range queries because multi-versioning does not bring any benefits to phantom prevention.
* Existing approaches to provide serializable transaction processing use either:
  1. additional latches in the index, or
  2. extra validation steps when transactions commit.

### 3.1. Timestamp Ordering (MVTO)

* Considered to be the original multi-version concurrency control protocol.
* The crux of this approach is to use the transactions’ identifiers (Tid) to precompute their serialization order.
* In addition to the fields described in Sect. 2.2, the version headers also contain the identifier of the last
  transaction that read it (read-ts). The DBMS aborts a transaction
  that attempts to read or update a version whose write lock is held by another transaction.
* Read operation
  * the DBMS searches for a physical version where Tid is in between the range of the begin-ts and end-ts fields.
  * T is allowed to read version Ax if its write lock is not held by another active transaction (i.e., value of txn-id
    is zero or equal to Tid) (never allows a transaction to read uncommitted)
  * Upon reading Ax, the DBMS sets Ax’s read-ts field to Tid if its current value is less than Tid. Otherwise, the
    transaction reads an older version without updating this field.

* Write operation:
  * Transaction T creates a new version Bx+1 if (1) no active transaction holds Bx’s write lock and (2) Tid is larger
    than Bx’s read-ts field.

* Commit Operation(?):
  * the DBMS sets Bx+1’s begin-ts and end-ts fields to Tid and INF (respectively), and Bx’s end-ts field to Tid.

* Example:

|              | Tuple | TNX-ID | READ_TS | BEGIN-TS | END-TS |
|--------------|-------|--------|---------|----------|--------|
| initial      | A1    | 0      | 0       | 1        | INF    |
| T2-write     | A1    | 2      | 0       | 1        | INF    |
|              | A2    | 2      | 0       | 2        | INF    |
| T2-commit(?) | A1    | 0      | 0       | 1        | 2      |
|              | A2    | 0      | 0       | 2        | INF    |
| T3-read      | A2    | 0      | 3       | 2        | INF    |

### 3.2 Optimistic Concurrency Control (MVOCC)

### 3.3 Two-phase Locking (MV2PL)

* In an in-memory DBMS the locks are embedded in the tuple headers.
* Write lock: the txn-id field.
* Read lock: read-cnt field to count the number of active transactions that have read the tuple.
* Read operation:
  * Search for a visible version by comparing Tid with begin-ts.
  * Increment the read-cnt if txn-id is zero (no other transaction holds the write lock).
* Write operation:
  * a transaction is allowed to update a version Bx only if both read-cnt and txn-id are set to zero.
* Commit Operation:
  * Assign a unique commit timestamp (Tcommit).
  * Update the begin-ts field for new versions.
  * Release all transaction locks.

* Example:

|              | Tuple | TNX-ID | READ_CNT | BEGIN-TS | END-TS |
|--------------|-------|--------|----------|----------|--------|
| initial      | A1    | 0      | 0        | 1        | INF    |
|              | B1    | 0      | 0        | 1        | INF    |
| T2-read      | A1    | 0      | 1        | 1        | INF    |
| T2-write     | B1    | 2      | 2        | 1        | INF    |
|              | B2    | 2      | 0        | 2        | INF    |
| T2-commit(?) | B1    | 0      | 0        | 1        | 2      |
|              | B2    | 0      | 0        | 2        | INF    |

## References

* [An Empirical Evaluation of In-Memory Multi-Version Concurrency Control](https://db.cs.cmu.edu/papers/2017/p781-wu.pdf)
* [15-721 Advanced Database Systems (Spring 2020) CMU Database Group](https://www.youtube.com/playlist?list=PLSE8ODhjZXjasmrEd2_Yi1deeE360zv5O)

