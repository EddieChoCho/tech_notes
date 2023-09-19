# Distributed Locks with Redis

* Redlock - a more canonical algorithm to implement distributed locks with Redis.

## Safety and Liveness Guarantees

* The minimum guarantees needed to use distributed locks in an effective way:
  * Safety property: Mutual exclusion. At any given moment, only one client can hold a lock.
  * Liveness property A: Deadlock free. Eventually it is always possible to acquire a lock, even if the client that
    locked a resource crashes or gets partitioned.
  * Liveness property B: Fault tolerance. As long as the majority of Redis nodes are up, clients are able to acquire and
    release locks.

## Why Failover-based Implementations Are Not Enough

* The simplest way to use Redis to lock a resource is to create a key in an instance.
  * The key is usually created with a limited time to live (property 2 in our list), using the Redis expires feature.
  * When the client needs to release the resource, it deletes the key.

* A single point of failure in our architecture.
  * If the Redis master goes down, adding a replica and use it if the master is unfortunately not viable.
  * Redis replication is asynchronous, we can’t implement our safety property of mutual exclusion by doing so.
  * A race condition with this model:
    1. Client A acquires the lock in the master.
    2. The master crashes before the write to the key is transmitted to the replica.
    3. The replica gets promoted to master.
    4. Client B acquires the lock to the same resource A already holds a lock for. SAFETY VIOLATION!

## Correct Implementation with a Single Instance

* To acquire the lock, the way to go is the following:
    ```
    set {$resource_name} {my_random_value} nx px {$timeout}
    ```
    * NX option: The command will set the key only if it does not already exist
    * PX option: Expire time (milliseconds)
    * Set to a value “my_random_value”, which must be unique across all clients and all lock requests, to the key.
    * Random value is used in order to release the lock in a safe way.
        * The key will be removed only if it exists and the value stored at the key is exactly the one I expect to be.
        * This can avoid removing a lock that was created by another client.

* To release the lock in a safe way(remove the key only if it exists and the value stored at the key is exactly the one
  I expect to be.):
  ```
  if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
  else
    return 0
  end
  ```
    * The Lua script can ensure the `atomic` of the release-lock operation.
    * How could it be atomic? https://redis.io/docs/manual/programmability/eval-intro/
    * Eval? https://redis.io/docs/manual/programmability/functions-intro/

* With this system, reasoning about a non-distributed system composed of a single, always available, instance, is safe.

## The Redlock Algorithm

* In the distributed version of the algorithm we assume we have N Redis masters which are totally independent.
* In our examples we set N=5, which is a reasonable value, so we need to run 5 Redis masters on different computers or
  virtual machines in order to ensure that they’ll fail in a mostly independent way.
* In order to acquire the lock, the client performs the following operations:

1. It gets the current time in milliseconds.
2. It tries to acquire the lock in all the N instances sequentially, using the same key name and random value in all the
   instances.
  * During step 2, when setting the lock in each instance, the client uses a timeout which is small compared to the
    total lock auto-release time in order to acquire it. For example if the auto-release time is 10 seconds, the timeout
    could be in the ~ 5-50 milliseconds range.
  * This prevents the client from remaining blocked for a long time trying to talk with a Redis node which is down: if
    an instance is not available, we should try to talk with the next instance ASAP.

3. The client computes how much time elapsed in order to acquire the lock, by subtracting from the current time the
   timestamp obtained in step 1. If and only if the client was able to acquire the lock in the majority of the
   instances (at least 3), and the total time elapsed to acquire the lock is less than lock validity time, the lock is
   considered to be acquired.
4. If the lock was acquired, its validity time is considered to be the initial validity time minus the time elapsed, as
   computed in step 3.
5. If the client failed to acquire the lock for some reason (either it was not able to lock N/2+1 instances or the
   validity time is negative), it will try to unlock all the instances (even the instances it believed it was not able
   to lock).

## Redisson

### Lock

* Lock watchdog
    * If Redisson instance which acquired lock crashes then such lock could hang forever in acquired state. To avoid
      this Redisson maintains lock watchdog.
    * It prolongs lock expiration while lock holder Redisson instance is alive.
    * By default, lock watchdog timeout is 30 seconds(can be changed through Config.lockWatchdogTimeout setting).

* `leaseTime`
    * A parameter during lock acquisition can be defined. After specified time interval locked lock will be released
      automatically.

* Code example:

```
RLock lock = redisson.getLock("myLock");

// traditional lock method
lock.lock();

// or acquire lock and automatically unlock it after 10 seconds
lock.lock(10, TimeUnit.SECONDS);

// or wait for lock aquisition up to 100 seconds 
// and automatically unlock it after 10 seconds
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
if (res) {
   try {
     ...
   } finally {
       lock.unlock();
   }
}
```

### How does RedissonRedLock implement Redlock Algorithm?[2]

* The RedissonRedLock object implements the Redlock locking algorithm for using distributed locks with Redis:
    ```
        RLock lock1 = redissonInstance1.getLock("lock1");
        RLock lock2 = redissonInstance2.getLock("lock2");
        RLock lock3 = redissonInstance3.getLock("lock3");
  
        RedissonRedLock lock = new RedissonRedLock(lock1, lock2, lock3);
  
        lock.lock();
        try {
            //...
        } finally {
            lock.unlock();
        }
    ```
* In the Redlock algorithm, we have a number of independent Redis master nodes located on separate computers or virtual
  machines.
* The algorithm attempts to acquire the lock in each of these instances sequentially, using the same key name and random
  value.
* The lock is only acquired if the client was able to acquire the lock from the majority of the instances quicker than
  the total time for which the lock is valid.

## References

* [1][Distributed Locks with Redis](https://redis.io/docs/reference/patterns/distributed-locks/)
* [2][Distributed Java Locks With Redis - DZone](https://dzone.com/articles/distributed-java-locks-with-redis)
