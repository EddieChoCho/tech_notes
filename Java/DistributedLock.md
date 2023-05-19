# Distributed lock

## Different kinds of distributed lock

* Distributed lock using database
* Distributed lock using memory cache
* Distributed lock using Zookeeper

## Distributed Locks with Spring Integration

* LockRegistry
* DefaultLockRepository
    * The default implementation of the LockRepository based on the table from the script presented in the
      org/springframework/integration/jdbc/schema-*.sql.
    ```
      CREATE TABLE INT_LOCK  (
      LOCK_KEY CHAR(36),
      REGION VARCHAR(100),
      CLIENT_ID CHAR(36),
      CREATED_DATE TIMESTAMP NOT NULL,
      constraint INT_LOCK_PK primary key (LOCK_KEY, REGION)
      );
    ```

    * This repository can't be shared between different JdbcLockRegistry instances. Otherwise, it opens a possibility to
      break Lock contract, where JdbcLockRegistry uses non-shared ReentrantLocks for local synchronizations.
* JdbcLockRegistry
    * An ExpirableLockRegistry using a shared database to co-ordinate the locks.
    * Provides the same semantics as the DefaultLockRegistry, but the locks taken will be global, as long as the
      underlying database supports the "serializable" isolation level in its transactions.
* RedisLockRegistry
    * Implementation of ExpirableLockRegistry providing a distributed lock using Redis.
    * Locks are stored under the key registryKey:lockKey.
    * Locks expire after (default 60) seconds. Threads unlocking an expired lock will get an IllegalStateException. This
      should be considered as a critical error because it is possible the protected resources were compromised.
* ZookeeperLockRegistry

### JdbcLockRegistry.JdbcLock

#### Fields

* private final LockRepository mutex;
* private final Duration idleBetweenTries;
* private final String path;
* private volatile long lastUsed = System.currentTimeMillis();
* private final ReentrantLock delegate = new ReentrantLock();

#### Methods

1. public void lock():
    1. lock the delegate(ReentrantLock)
    2. keep trying to `this.mutex.acquire(this.path);`
        * If acquire failed, `Thread.sleep(this.idleBetweenTries.toMillis());` then retry
        * If acquired, update the `lastUsed`.
            * mutex.acquire:
                * `@Transactional(propagation = Propagation.REQUIRES_NEW, isolation = Isolation.SERIALIZABLE)`
                * "UPDATE %sLOCK SET CLIENT_ID=?, CREATED_DATE=? WHERE REGION=? AND LOCK_KEY=? AND (CLIENT_ID=? OR
                  CREATED_DATE<?)" or
                * "INSERT INTO %sLOCK (REGION, LOCK_KEY, CLIENT_ID, CREATED_DATE) VALUES (?, ?, ?, ?)"
                * catch DuplicateKeyException, return false;


2. public void unlock():
    1. check if `this.delegate.isHeldByCurrentThread()`
    2. if (this.delegate.getHoldCount() > 1): this.delegate.unlock();
    3. else `this.mutex.delete(this.path);`
        * mutex.delete
            * @Transactional(propagation = Propagation.REQUIRES_NEW)
            * "DELETE FROM %sLOCK WHERE REGION=? AND LOCK_KEY=? AND CLIENT_ID=?";

3. public boolean tryLock(long time, TimeUnit unit):
    1. `delegate.tryLock(time, unit)`(ReentrantLock)
    2. keep trying to `this.mutex.acquire(this.path);` and checking if timeout
        * If acquire failed, `Thread.sleep(this.idleBetweenTries.toMillis());` then retry
        * If acquired, update the `lastUsed`.

4. public boolean tryLock():
    * `tryLock(0, TimeUnit.MICROSECONDS);`

5. public boolean isAcquiredInThisProcess():
    * `this.mutex.isAcquired(this.path);`
        * @Transactional(propagation = Propagation.REQUIRES_NEW, readOnly = true)
        * "SELECT COUNT(REGION) FROM %sLOCK WHERE REGION=? AND LOCK_KEY=? AND CLIENT_ID=? AND CREATED_DATE>=?"; == 1

### RedisLockRegistry.RedisLock(RedisSpinLock)

#### Methods

* public final boolean tryLock(long time, TimeUnit unit)
    * `this.localLock.tryLock(time, unit)`
    * protected final Boolean obtainLock():

      KEYS:    `Collections.singletonList(this.lockKey)`, ARGV[1]:    `RedisLockRegistry.this.clientId`,
      ARGV[2]:    `String.valueOf(RedisLockRegistry.this.expireAfter)`

      ```
      local lockClientId = redis.call('GET', KEYS[1])
          if lockClientId == ARGV[1] then
              redis.call('PEXPIRE', KEYS[1], ARGV[2])
              return true
          elseif not lockClientId then
              redis.call('SET', KEYS[1], ARGV[1], 'PX', ARGV[2])
              return true
          end 
              return false
      ```

      PEXPIRE: Set a timeout on key. After the timeout has expired, the key will automatically be deleted.

* public final void unlock()
    * if `this.localLock.isHeldByCurrentThread()`
    * if `this.localLock.getHoldCount() > 1`
        * `this.localLock.unlock(); return;`
    * else
        *
        if `RedisLockRegistry.this.clientId.equals(RedisLockRegistry.this.redisTemplate.boundValueOps(this.lockKey).get())`
        //checking if value is equals to clientId
            * `RedisLockRegistry.this.redisTemplate.unlink(this.lockKey)`;
            * `RedisLockRegistry.this.redisTemplate.delete(this.lockKey)`;

## References

* [Spring Tips: Distributed Locks with Spring Integration](https://youtu.be/firwCHbC7-c)
* [JDBC Lock Registry](https://docs.spring.io/spring-integration/reference/html/jdbc.html#jdbc-lock-registry)
