# How does Spring Distributed Lock work? Exploring source code of RedisLockRegistry and RedisSpinLock

In this article, we embark on a journey to unravel the intricate mechanisms of Spring Distributed Lock using Redis.

With a keen focus on source code analysis, this article unveils essential insights into the architecture of
RedisLockRegistry. It sheds light on crucial design considerations that drive the creation of a distributed lock
solution within a single Redis node environment.

If you're new to the concept of Spring Distributed Lock, consider exploring this informative
article: [How To Implement a Spring Distributed Lock](https://tanzu.vmware.com/developer/guides/spring-integration-lock/)
, or watch this video: [Spring Tips: Distributed Locks with Spring Integration](https://youtu.be/firwCHbC7-c), to
familiarize yourself before delving into the content presented here.

The Spring Integration project offers a range of distributed lock options. we will delve into the implementation using
Redis, specifically through RedisLockRegistry.java.

RedisLock is an abstract inner class of RedisLockRegistry class, the instance of RedisLock will be used to manage
concurrent access by insert, update, or delete data, the status of the lock, in Redis service. RedisSpinLock and
RedisPubSubLock are the classes that inherit from the RedisLock abstract class. Our focus in this article will be on the
former, RedisSpinLock.

In this section, we will explore the source code
of [RedisLockRegistry.java(version 6.1.x)]( https://github.com/spring-projects/spring-integration/blob/6.1.x/spring-integration-redis/src/main/java/org/springframework/integration/redis/util/RedisLockRegistry.java
) in detail, dissecting its components to comprehend the underlying mechanism of this distributed lock.

### The Contraction of RedisLockRegistry bean

```java
public final class RedisLockRegistry {
    //...
    private static final long DEFAULT_EXPIRE_AFTER = 60000L;

    private final String clientId = UUID.randomUUID().toString();

    private final String registryKey;
    //...
    private final StringRedisTemplate redisTemplate;

    private final long expireAfter;

    //...
    public RedisLockRegistry(RedisConnectionFactory connectionFactory, String registryKey) {
        this(connectionFactory, registryKey, DEFAULT_EXPIRE_AFTER);
    }

    public RedisLockRegistry(RedisConnectionFactory connectionFactory, String registryKey, long expireAfter) {
        Assert.notNull(connectionFactory, "'connectionFactory' cannot be null");
        Assert.notNull(registryKey, "'registryKey' cannot be null");
        this.redisTemplate = new StringRedisTemplate(connectionFactory);
        this.registryKey = registryKey;
        this.expireAfter = expireAfter;
        this.unLockChannelKey = registryKey + "-channel";
    }
    //...
}
```

When creating a RedisLockRegistry instance, you need to provide three parameters: `connectionFactory`, `registryKey`,
and `expireAfter`.

The `connectionFactory` parameter is used to generate the redisTemplate instance field within the RedisLockRegistry.
This redisTemplate facilitates interaction with Redis using string values.

The `registryKey` parameter serves as a prefix for key names. These key names are formed by combining the `registryKey`
and the key's specific value, separated by a colon. For example, if `registryKey` is set to "Spring" and you intend to
lock a key with the value "Integration", the actual key used to communicate with Redis would be "Spring:Integration".

The `expireAfter` parameter, of type long, determines the Time To Live (TTL) for the lock status data inserted into
Redis by RedisLock instances.

An important point to note is the `clientId` instance field. This is a UUID string generated during the creation of a
RedisLockRegistry object. It represents the process's identity and is used as the value associated with each key. Which
will be used to veirfy the ownership of the distributed lock. As a result, each process should maintain only one
instance of RedisLockRegistry.

### Obtaining Locks from RedisLockRegistry

```java
public final class RedisLockRegistry {
    //...
    private static final int DEFAULT_CAPACITY = 100_000;

    private int cacheCapacity = DEFAULT_CAPACITY;

    private RedisLockType redisLockType = RedisLockType.SPIN_LOCK;

    private final Lock lock = new ReentrantLock();

    private final Map<String, RedisLock> locks = new LinkedHashMap<>(16, 0.75F, true) {
        @Override
        protected boolean removeEldestEntry(Entry<String, RedisLock> eldest) {
            return size() > RedisLockRegistry.this.cacheCapacity;
        }
    };

    //...
    @Override
    public Lock obtain(Object lockKey) {
        Assert.isInstanceOf(String.class, lockKey);
        String path = (String) lockKey;
        this.lock.lock();
        try {
            return this.locks.computeIfAbsent(path, getRedisLockConstructor(this.redisLockType));
        } finally {
            this.lock.unlock();
        }
    }

    @Override
    public void expireUnusedOlderThan(long age) {
        long now = System.currentTimeMillis();
        this.lock.lock();
        try {
            this.locks.entrySet()
                    .removeIf(entry -> {
                        RedisLock lock = entry.getValue();
                        long lockedAt = lock.getLockedAt();
                        return now - lockedAt > age
                                // 'lockedAt = 0' means that the lock is still not acquired!
                                && lockedAt > 0
                                && !lock.isAcquiredInThisProcess();
                    });
        } finally {
            this.lock.unlock();
        }
    }
    //...
}
```

To initiate the acquisition of a distributed lock, the initial step is to obtain a RedisLock object from the
RedisLockRegistry. This process can be accomplished by invoking the `obtain` method on a RedisLockRegistry instance,
passing in a string key.

In the source code of the `obtain` method, the ReentrantLock instance field of RedisLockRegistry - `lock` is pivotal in
ensuring the orderly acquisition of a RedisLock. Its purpose is to restrict only one thread at a time from attempting to
acquire a RedisLock. Should no RedisLock object be associated with the provided key in the `locks` map, a new RedisLock
is created with the key and returned. The value of the value will be assign to the String instance variable `lockKey` of
RedisLock. By default, this newly created RedisLock is an instance of RedisSpinLock. On the other hand, if a RedisLock
object already exists for the key, that existing instance will be returned.

Furthermore, the `lock` instance field also manages concurrent access to the `locks` map in the `expireUnusedOlderThan`
method. Consequently, there is no need to be concerned about potential race conditions between these two methods.

### Acquiring Distributed Lock through Invoking tryLock method of RedisLock

```java
public final class RedisLockRegistry {
    //...
    private abstract class RedisLock implements Lock {
        //...
        private final ReentrantLock localLock = new ReentrantLock();

        //....
        @Override
        public final boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
            if (!this.localLock.tryLock(time, unit)) {
                return false;
            }
            try {
                long waitTime = TimeUnit.MILLISECONDS.convert(time, unit);
                // Try to insert/update the status of lock into Redis
                boolean acquired = tryRedisLock(waitTime);
                if (!acquired) {
                    this.localLock.unlock();
                }
                return acquired;
            } catch (Exception e) {
                this.localLock.unlock();
                rethrowAsLockException(e);
            }
            return false;
        }
        //...
    }
    //...
}
```

Once the RedisLock instance is obtained, we can proceed to invoke the `tryLock` method. This call attempts to acquire
the distributed lock from Redis. It's noteworthy that RedisLock is equipped with its own ReentrantLock instance,
referred to as `localLock`. This local lock ensures the thread safety of using the RedisLock.

At the outset of the `tryLock` method, the `localLock` serves to ensure that only one thread can execute lock-related
methods of the same RedisLock instance simultaneously.

Subsequently, the `tryRedisLock` method is executed to make an attempt at acquiring the lock from Redis, with a defined
waiting time. If the lock is successfully acquired, the `localLock` is unlocked and true is returned. Conversely, if the
acquisition fails or any exceptions occur during the process, the `localLock` is still unlocked to prevent potential
deadlocks, and false is returned to signal the unsuccessful acquisition to the caller.

The `tryRedisLock` method internally invokes the tryRedisLockInner method, an abstract method with distinct
implementations in RedisSpinLock and RedisPubSubLock. For an in-depth exploration of this process, we'll now delve into
the implementation within RedisSpinLock to understand the specific actions undertaken.

```java
public final class RedisLockRegistry {
    //...
    private final class RedisSpinLock extends RedisLock {
        //...
        @Override
        protected boolean tryRedisLockInner(long time) throws InterruptedException {
            long now = System.currentTimeMillis();
            if (time == -1L) {
                while (!obtainLock()) {
                    Thread.sleep(100);
                }
                return true;
            } else {
                long expire = now + TimeUnit.MILLISECONDS.convert(time, TimeUnit.MILLISECONDS);
                boolean acquired;
                while (!(acquired = obtainLock()) && System.currentTimeMillis() < expire) {
                    Thread.sleep(100);
                }
                return acquired;
            }
        }
    }
}
```

The `tryRedisLockInner` method diligently attempts to invoke the `obtainLock` method every 100 milliseconds, which
handles the communication task with Redis. When `time` is set to -1, indicating that the waiting time hasn't been
specified, the `tryRedisLockInner` method perseveres until the `obtainLock` method returns true. Alternatively, when a
waiting time is specified, the method continuously checks if the waiting time has been exceeded. In either case, the
method returns the acquisition result, whether successful or unsuccessful.

In the pursuit of reliability, the `tryRedisLockInner` method showcases a robust structure that supports both indefinite
and time-bound lock acquisition scenarios.

```java
public final class RedisLockRegistry {
    // The id of the process
    private final String clientId = UUID.randomUUID().toString();

    //...
    private abstract class RedisLock implements Lock {

        private static final String OBTAIN_LOCK_SCRIPT = """
                local lockClientId = redis.call('GET', KEYS[1])
                if lockClientId == ARGV[1] then
                    redis.call('PEXPIRE', KEYS[1], ARGV[2])
                    return true
                elseif not lockClientId then
                    redis.call('SET', KEYS[1], ARGV[1], 'PX', ARGV[2])
                    return true
                end
                    return false
                    """;
        protected static final RedisScript<Boolean>
                OBTAIN_LOCK_REDIS_SCRIPT = new DefaultRedisScript<>(OBTAIN_LOCK_SCRIPT, Boolean.class);

        protected final String lockKey;

        protected final Boolean obtainLock() {
            return RedisLockRegistry.this.redisTemplate
                    .execute(OBTAIN_LOCK_REDIS_SCRIPT, Collections.singletonList(this.lockKey),
                            RedisLockRegistry.this.clientId,
                            String.valueOf(RedisLockRegistry.this.expireAfter));
        }
        //....
    }
}
```

The `obtainLock` method carries out a Lua script, OBTAIN_LOCK_SCRIPT, utilizing the `EVALSHA` command. This script
involves three components: the `lockKey`, which was assigned a value during the instantiation of RedisLock; `clientId`,
and `expireAfter`, both derived from the constructor of RedisLockRegistry.

Upon execution of the Lua script, the following operations occur within Redis:

1. Check if a value exists for `lockKey` using the `GET` operation.
2. If there's no existing value, implying the distributed lock for this key hasn't been acquired by any process, execute
   a `SET` operation. This inserts data into Redis with the key, value (clientId), and a TTL specified by `expireAfter`.
   The TTL will function as a safeguard against potential deadlocks.
3. If the value retrieved from the `GET` operation matches `clientId`, it indicates the current process has previously
   acquired the distributed lock for this key and it's still locked. In this case, execute a `PEXPIRE` operation to
   update the lock's timeout using the `expireAfter` value.
4. If the value from the `GET` operation differs from `clientId`, indicating the distributed lock is currently held by
   another process, the script returns false to signify an unsuccessful acquisition.

It's important to note that although the script may execute multiple commands, Redis guarantees atomic execution of
scripts and functions, eliminating concerns about concurrency issues.

The method showcases a well-structured script execution process within Redis, ensuring the proper acquisition and
management of distributed locks.

### Releasing a Distributed Lock by Invoking the unlock Method of RedisLock

```java
public final class RedisLockRegistry {
    //...
    private abstract class RedisLock implements Lock {
        //...
        private final ReentrantLock localLock = new ReentrantLock();

        //...
        @Override
        public final void unlock() {
            if (!this.localLock.isHeldByCurrentThread()) {
                throw new IllegalStateException("You do not own lock at " + this.lockKey);
            }
            if (this.localLock.getHoldCount() > 1) {
                this.localLock.unlock();
                return;
            }
            try {
                // Remove the status of lock from Redis
                if (Thread.currentThread().isInterrupted()) {
                    RedisLockRegistry.this.executor.execute(this::removeLockKey);
                } else {
                    removeLockKey();
                }

                if (LOGGER.isDebugEnabled()) {
                    LOGGER.debug("Released lock; " + this);
                }
            } catch (Exception e) {
                ReflectionUtils.rethrowRuntimeException(e);
            } finally {
                this.localLock.unlock();
            }
        }
        //...
    }
    //...
}
```

Upon successfully acquiring the distributed lock and completing execution within the critical section, it's imperative
to release the lock. This is achieved by invoking the unlock method on the RedisLock object.

Within the execution of the `unlock` method, `isHeldByCurrentThread` of the `localLock` is invoked to verify if the
RedisLock is locked by the current thread. If another thread holds the lock, the current thread must not proceed with
unlocking. Consequently, an IllegalStateException is raised.

Given that the `localLock`  is an instance of ReentrantLock, it's feasible for a thread to hold the lock through
multiple locking actions that aren't necessarily matched by an equal number of unlocking actions. To account for this,
the hold count of the thread is examined:

If the hold count surpasses one, signifying that additional unlock actions must be executed before releasing the
distributed lock, the `unlock` method of `localLock` is invoked, and the method's execution concludes.

Conversely, if the hold count equals one, denoting that the distributed lock can be released, the `removeLockKey` method
is called within a try block. This method is responsible for erasing the lock's status—previously inserted into Redis by
the `obtainLock` method of RedisLock. The final invocation of `localLock.unlock` within the current lifecycle of
distributed lock utilization takes place within the finally block. This ensures that regardless of exceptions or flow
control, the lock is reliably released.

```java
public final class RedisLockRegistry {
    //...
    private abstract class RedisLock implements Lock {
        //...
        private void removeLockKey() {
            if (RedisLockRegistry.this.unlinkAvailable) {
                Boolean unlinkResult = null;
                try {
                    // Attempt to UNLINK the lock key; an exception indicates lack of UNLINK support
                    unlinkResult = removeLockKeyInnerUnlink();
                } catch (Exception ex) {
                    RedisLockRegistry.this.unlinkAvailable = false;
                    if (LOGGER.isDebugEnabled()) {
                        LOGGER.debug("The UNLINK command has failed (not supported on the Redis server?); " +
                                "falling back to the regular DELETE command", ex);
                    } else {
                        LOGGER.warn("The UNLINK command has failed (not supported on the Redis server?); " +
                                "falling back to the regular DELETE command: " + ex.getMessage());
                    }
                }

                if (Boolean.TRUE.equals(unlinkResult)) {
                    // Lock key successfully unlinked
                    return;
                } else if (Boolean.FALSE.equals(unlinkResult)) {
                    throw new IllegalStateException("Lock was released in the store due to expiration. " +
                            "The integrity of data protected by this lock may have been compromised.");
                }
            }
            if (!removeLockKeyInnerDelete()) {
                throw new IllegalStateException("Lock was released in the store due to expiration. " +
                        "The integrity of data protected by this lock may have been compromised.");
            }
        }
    }
}
```

During the execution of the `removeLockKey` method, the `unlinkAvailable` flag serves as a determinant for choosing
between invoking `removeLockKeyInnerUnlink` or `removeLockKeyInnerDelete` methods to eliminate the lock's status.

By default, the value of `unlinkAvailable` is set to true, which triggers the execution of
the `removeLockKeyInnerUnlink` method. If an exception occurs during the execution of `removeLockKeyInnerUnlink`, it
indicates that the Redis service does not support the `UNLINK` operation. Consequently, the `unlinkAvailable` flag is
switched to false, subsequently leading to the execution of the `removeLockKeyInnerDelete` method. This method employs
the `DEL` command to perform the deletion of the lock's status from Redis.

When the `removeLockKeyInnerUnlink` method returns a true value, it confirms the successful release of the distributed
lock. Conversely, a false value indicates that the lock has either expired or is held by another process, prompting the
generation of an IllegalStateException.

The `removeLockKey` method exemplifies an essential aspect of the distributed lock's lifecycle—safely releasing the
lock's hold on shared resources while accounting for potential exceptions and variations in Redis service capabilities.

```java
public final class RedisLockRegistry {
    //...
    private abstract class RedisSpinLock implements Lock {

        private static final String UNLINK_UNLOCK_SCRIPT = """
                local lockClientId = redis.call('GET', KEYS[1])
                if lockClientId == ARGV[1] then
                    redis.call('UNLINK', KEYS[1])
                    return true
                 end
                 return false
                 """;
        //...
        private static final RedisScript<Boolean>
                UNLINK_UNLOCK_REDIS_SCRIPT = new DefaultRedisScript<>(UNLINK_UNLOCK_SCRIPT, Boolean.class);

        //...
        @Override
        protected boolean removeLockKeyInnerUnlink() {
            return removeLockKeyWithScript(UNLINK_UNLOCK_REDIS_SCRIPT);
        }

        //...
        private boolean removeLockKeyWithScript(RedisScript<Boolean> redisScript) {
            return Boolean.TRUE.equals(RedisLockRegistry.this.redisTemplate.execute(
                    redisScript, Collections.singletonList(this.lockKey),
                    RedisLockRegistry.this.clientId));
        }
    }
    //...
}
```

The `removeLockKeyInnerUnlink` method carries out a Lua script named UNLINK_UNLOCK_SCRIPT by utilizing the `EVALSHA`
command. This script involves two components: the `lockKey` and `clientId`.

Upon executing the Lua script, the following Redis operations transpire:

1. A `GET` operation checks if a value exists for the `lockKey`.
2. If the retrieved value matches the `clientId`, signifying that the current process holds the distributed lock,
   an `UNLINK` operation is executed. This operation removes the lock's status.
3. If the value from the `GET` operation differs from the `clientId`, indicating that the distributed lock is currently
   held by another process, the script returns false, indicating an unsuccessful release.
4. When there's no existing value for the `lockKey`, indicating the distributed lock has expired, the script also
   returns false to indicate an unsuccessful release.

The `removeLockKeyInnerDelete` method features a nearly identical implementation to `removeLockKeyInnerUnlink`. The sole
distinction lies in the script executed by `removeLockKeyInnerDelete`, which employs the `DEL` command as opposed to
the `UNLINK` command.

### Summary

Central to this solution desgin of RedisLockRegistry is the utilization of the lock construct, specifically the
ReentrantLock. This essential component ensures the thread-safe characteristic of the distributed lock within a single
process. By encapsulating critical sections of code with these locks, we guarantee that only one thread gains access at
a time, mitigating potential conflicts that arise from concurrent execution.

However, the scope of distributed locks extends beyond the boundaries of a single process. The solution integrates the
concept of atomic execution. By orchestrating operations such as lock acquisition and release with atomic execution, the
solution prevents concurrency issues across processes.

## References

* [Spring Integration](https://github.com/spring-projects/spring-integration)
* [RedisLockRegistry.java(6.1.x version)](https://github.com/spring-projects/spring-integration/blob/6.1.x/spring-integration-redis/src/main/java/org/springframework/integration/redis/util/RedisLockRegistry.java)
* [How To Implement a Spring Distributed Lock](https://tanzu.vmware.com/developer/guides/spring-integration-lock/)
* [Distributed Locks with Redis](https://redis.io/docs/manual/patterns/distributed-locks/)
* [Redis programmability](https://redis.io/docs/interact/programmability/)