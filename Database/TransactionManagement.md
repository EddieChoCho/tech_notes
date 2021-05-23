# Transaction Management: Concurrency Control

 * Consistency
 * Isolation
 	* Interleaved execution of txs should have the net effect identical to executing tx in some serial order.
 	* T1 and T2 are executed concurrently, isolation gives that the net effect to be equivalent to either T1 followed by T2 ot T2 followed by T1.
 	* The DBMS does not guarantee the result in which particular order.

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
* Stric schedules
	* A schedule is strict if for any two txs T1 and T2, if a write operation of T1 precedes a conflicting operation of T2(either read or write), then Y1 commits before that conflicting operation of T2.
	* Avoids cascading rollback, but still has deadlock.

### Deadlock
* Detect `Waits-for` graph when acquiring locks(or buffers)
* Otehr techniques
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

# References
* [Introduction to Database System by Shan-Hung Wu](https://www.youtube.com/playlist?list=PLS0SUwlYe8cyln89Srqmmlw42CiCBT6Zn)
