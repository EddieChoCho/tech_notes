## Distributed lock
### Different kinds of distributed lock 
* Distributed lock using database
* Distributed lock using memory cache
* Distributed lock using Zookeeper

### Distributed Locks with Spring Integration
* LockRegistry
* DefaultLockRepository
	* The default implementation of the LockRepository based on the table from the script presented in the org/springframework/integration/jdbc/schema-*.sql.
		* INT_LOCK table?
	* This repository can't be shared between different JdbcLockRegistry instances. Otherwise it opens a possibility to break Lock contract, where JdbcLockRegistry uses non-shared ReentrantLocks for local synchronizations.
* JdbcLockRegistry
	* An ExpirableLockRegistry using a shared database to co-ordinate the locks. 
	* Provides the same semantics as the DefaultLockRegistry, but the locks taken will be global, as long as the underlying database supports the "serializable" isolation level in its transactions.
* RedisLockRegistry
	* Implementation of ExpirableLockRegistry providing a distributed lock using Redis. 
	* Locks are stored under the key registryKey:lockKey. 
	* Locks expire after (default 60) seconds. Threads unlocking an expired lock will get an IllegalStateException. This should be considered as a critical error because it is possible the protected resources were compromised.
* ZookeeperLockRegistry

## References
* [Spring Tips: Distributed Locks with Spring Integration](https://youtu.be/firwCHbC7-c)
