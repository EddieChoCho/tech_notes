# Introduction to Spring Distributed Lock(Using RedisLockRegistry & RedisSpinLock as Example)

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

	To initiate the acquisition of a distributed lock, the initial step is to obtain a RedisLock object from the RedisLockRegistry. This process can be accomplished by invoking the `obtain` method on a RedisLockRegistry instance, passing in a string key.

In the source code of the `obtain` method, the `lock` instance field is pivotal in ensuring the orderly acquisition of a
RedisLock. Its purpose is to restrict only one thread at a time from attempting to acquire a RedisLock. Should no
RedisLock object be associated with the provided key in the `locks` map, a new RedisLock is created with the key and
returned. By default, this newly created RedisLock is an instance of RedisSpinLock. On the other hand, if a RedisLock
object already exists for the key, that existing instance will be returned.

Furthermore, the `lock` instance field also manages concurrent access to the `locks` map in the `expireUnusedOlderThan`
method. Consequently, there is no need to be concerned about potential race conditions between these two methods.

# References

* [How To Implement a Spring Distributed Lock](https://tanzu.vmware.com/developer/guides/spring-integration-lock/)
* [Distributed Locks with Redis](https://redis.io/docs/manual/patterns/distributed-locks/)
