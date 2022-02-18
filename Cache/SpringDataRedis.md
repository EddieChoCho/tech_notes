# Spring Data Redis

## Configuration

* JedisConnectionFactory
* RedisTemplate

## Working with Objects through RedisTemplate

* RedisTemplate v.s. RedisConnection
    * RedisConnection
        * RedisConnection offers low-level methods that accept and return binary values (byte arrays).
    * RedisTemplate
        * The template offers a high-level abstraction for Redis interactions.
        * The template takes care of serialization and connection management, freeing the user from dealing with such
          details.

* The template provides operations views (following the grouping from the Redis command reference).
    * e.g., ListOperations, SetOperations, ValueOperations...

* enableDefaultSerializer: You can also set any of the serializers to null and use RedisTemplate with raw byte arrays by
  setting the enableDefaultSerializer property to false.
* The template requires all keys to be non-null. However, values can be null as long as the underlying serializer
  accepts them.

## References

* [Spring Data Redis](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#reference)
