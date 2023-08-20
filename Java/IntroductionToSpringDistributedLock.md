# Introduction to Spring Distributed Lock with Redis(Using RedisLockRegistry & RedisSpinLock as Example)

## When do we need to use distributed locks?

Similar to using the `synchronized` keyword or Java Lock instances to control concurrent access to shared resources
within a single process, distributed locks serve a similar purpose but extend their reach to manage concurrent access
across multiple processes.

## How to use the Spring Distributed Lock?

<script src="https://gist.github.com/EddieChoCho/9b45bf5208c224fe8e276a072ca0d007.js?file=ExampleService.java"></script>

The `tryLock()` method can be invoked with two parameters: time (the maximum time to wait for the lock) and unit (the
time unit of the time argument). This prevents indefinite blocking and sets a maximum waiting time. In the code example,
the method is executed with a timeout of 6 minutes. Alternatively, if you are comfortable with blocking the thread until
the lock is acquired, you can use the `lock()` method. The placement of `unlock()` invocation within the finally block
ensures the consistent release of the lock, regardless of exceptions that may occur during execution. This guarantees
that even if an exception arises when executing the `tryLock()` method, the local lock of the distributed lock instance
is properly released, preventing potential thread blocking. You can find more details of using Spring Distributed Lock
with this
article: [How To Implement a Spring Distributed Lock](https://tanzu.vmware.com/developer/guides/spring-integration-lock/)

## How does it work?

The Spring Integration project offers a range of distributed lock options. In this article, we will delve into the
implementation using Redis, specifically through RedisLockRegistry.java. RedisLock is an abstract inner class of
RedisLockRegistry class, the instance of RedisLock will be used to manage concurrent access by insert, update, or delete
data, the status of the lock, in Redis service. RedisSpinLock and RedisPubSubLock are the classes that inherit from the
RedisLock abstract class. Our focus in this article will be on the former, RedisSpinLock. In this section, we will
explore the source code of RedisLockRegistry.java (version
6.1.x https://github.com/spring-projects/spring-integration/blob/6.1.x/spring-integration-redis/src/main/java/org/springframework/integration/redis/util/RedisLockRegistry.java
) in detail, dissecting its components to comprehend the underlying mechanism of this distributed lock.

### The Contraction of RedisLockRegistry bean

<script src="https://gist.github.com/EddieChoCho/9b45bf5208c224fe8e276a072ca0d007.js?file=Redislockregistry_construction.java/
"></script>

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
RedisLockRegistry object. It represents the process's identity and is used as the value associated with each key. As a
result, each process should maintain only one instance of RedisLockRegistry.

### Obtaining Locks from RedisLockRegistry

<script src="https://gist.github.com/EddieChoCho/9b45bf5208c224fe8e276a072ca0d007.js?file=Redislockregistry.java/"></script>

To initiate the acquisition of a distributed lock, the initial step is to obtain a RedisLock object from the
RedisLockRegistry. This process can be accomplished by invoking the `obtain` method on a RedisLockRegistry instance,
passing in a string key.

In the source code of the `obtain` method, the `lock` instance field is pivotal in ensuring the orderly acquisition of a
RedisLock. Its purpose is to restrict only one thread at a time from attempting to acquire a RedisLock. Should no
RedisLock object be associated with the provided key in the `locks` map, a new RedisLock is created with the key and
returned. By default, this newly created RedisLock is an instance of RedisSpinLock. On the other hand, if a RedisLock
object already exists for the key, that existing instance will be returned.

Furthermore, the `lock` instance field also manages concurrent access to the `locks` map in the `expireUnusedOlderThan`
method. Consequently, there is no need to be concerned about potential race conditions between these two methods.

### Acquiring Distributed Lock through Invoking tryLock method of RedisLock

<script src="https://gist.github.com/EddieChoCho/9b45bf5208c224fe8e276a072ca0d007.js?file=Redislock_trylock.java/"></script>

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

<script src="https://gist.github.com/EddieChoCho/9b45bf5208c224fe8e276a072ca0d007.js?file=Redisspinlock_tryredislockinner.java/"></script>

The `tryRedisLockInner` method diligently attempts to invoke the `obtainLock` method every 100 milliseconds, which
handles the communication task with Redis. When `time` is set to -1, indicating that the waiting time hasn't been
specified, the `tryRedisLockInner` method perseveres until the `obtainLock` method returns true. Alternatively, when a
waiting time is specified, the method continuously checks if the waiting time has been exceeded. In either case, the
method returns the acquisition result, whether successful or unsuccessful. In the pursuit of reliability,
the `tryRedisLockInner` method showcases a robust structure that supports both indefinite and time-bound lock
acquisition scenarios.


<script src="https://gist.github.com/EddieChoCho/9b45bf5208c224fe8e276a072ca0d007.js?file=RedisLock_obtainLock.java/"></script>

The `obtainLock` method carries out a Lua script, OBTAIN_LOCK_SCRIPT, utilizing the `EVALSHA` command. This script
involves three components: the `lockKey`, which was assigned a value during the instantiation of RedisLock; `clientId`,
and `expireAfter`, both derived from the constructor of RedisLockRegistry.

Upon execution of the Lua script, the following operations occur within Redis:

Check if a value exists for `lockKey` using the `GET` operation. If there's no existing value, implying the distributed
lock for this key hasn't been acquired by any process, execute a `SET` operation. This inserts data into Redis with the
key, value (clientId), and a TTL specified by `expireAfter`. The TTL will function as a safeguard against potential
deadlocks. If the value retrieved from the `GET` operation matches `clientId`, it indicates the current process has
previously acquired the distributed lock for this key and it's still locked. In this case, execute a `PEXPIRE` operation
to update the lock's timeout using the `expireAfter` value. If the value from the `GET` operation differs
from `clientId`, indicating the distributed lock is currently held by another process, the script returns false to
signify an unsuccessful acquisition.

It's important to note that although the script may execute multiple commands, Redis guarantees atomic execution of
scripts and functions, eliminating concerns about concurrency issues.

The method showcases a well-structured script execution process within Redis, ensuring the proper acquisition and
management of distributed locks.

### Releasing a Distributed Lock by Invoking the unlock Method of RedisLock

<script src="https://gist.github.com/EddieChoCho/9b45bf5208c224fe8e276a072ca0d007.js?file=Redislock_unlock.java/"></script>

Upon successfully acquiring the distributed lock and completing execution within the critical section, it's imperative
to release the lock. This is achieved by invoking the unlock method on the RedisLock object.

Within the execution of the `unlock` method, `isHeldByCurrentThread` of the `localLock` is invoked to verify if the
RedisLock is locked by the current thread. If another thread holds the lock, the current thread must not proceed with
unlocking. Consequently, an IllegalStateException is raised.

Given that the `localLock`  is an instance of ReentrantLock, it's feasible for a thread to hold the lock through
multiple locking actions that aren't necessarily matched by an equal number of unlocking actions. To account for this,
the hold count of the thread is examined:

If the hold count surpasses one, signifying that additional unlock actions must be executed before releasing the
distributed lock, the `unlock` method of `localLock` is invoked, and the method's execution concludes. Conversely, if
the hold count equals one, denoting that the distributed lock can be released, the `removeLockKey` method is called
within a try block. This method is responsible for erasing the lock's status—previously inserted into Redis by
the `obtainLock` method of RedisLock. The final invocation of `localLock.unlock` within the current lifecycle of
distributed lock utilization takes place within the finally block. This ensures that regardless of exceptions or flow
control, the lock is reliably released.

<script src="https://gist.github.com/EddieChoCho/9b45bf5208c224fe8e276a072ca0d007.js?file=Redislock_removelockkey.java/"></script>

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


<script src="https://gist.github.com/EddieChoCho/9b45bf5208c224fe8e276a072ca0d007.js?file=Redisspinlock_removelockkeyinnerunlink.java/"></script>


The `removeLockKeyInnerUnlink` method carries out a Lua script named UNLINK_UNLOCK_SCRIPT by utilizing the `EVALSHA`
command. This script involves two components: the `lockKey` and `clientId`.

Upon executing the Lua script, the following Redis operations transpire:

A` GET` operation checks if a value exists for the `lockKey`. If the retrieved value matches the `clientId`, signifying
that the current process holds the distributed lock, a `UNLINK` operation is executed. This operation removes the lock's
status. If the value from the `GET` operation differs from the `clientId`, indicating that the distributed lock is
currently held by another process, the script returns false, indicating an unsuccessful release. When there's no
existing value for the `lockKey`, indicating the distributed lock has expired, the script also returns false to indicate
an unsuccessful release.

The `removeLockKeyInnerDelete` method features a nearly identical implementation to `removeLockKeyInnerUnlink`. The sole
distinction lies in the script executed by `removeLockKeyInnerDelete`, which employs the `DEL` command as opposed to
the `UNLINK` command.

### References

* [Spring Integration](https://github.com/spring-projects/spring-integration)
* [How To Implement a Spring Distributed Lock](https://tanzu.vmware.com/developer/guides/spring-integration-lock/)
* [Distributed Locks with Redis](https://redis.io/docs/manual/patterns/distributed-locks/)
* [Redis programmability](https://redis.io/docs/interact/programmability/)

