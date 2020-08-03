## Hibernate Mapping

* constrained (optional): specifies that a foreign key constraint on the primary key of the mapped table and references the table of the associated class. This option affects the order in which save() and delete() are cascaded, and determines whether the association can be proxied. It is also used by the schema export tool.
* fetch (optional - defaults to select): chooses between outer-join fetching or sequential select fetching.
* property-ref (optional): the name of a property of the associated class that is joined to the primary key of this class. If not specified, the primary key of the associated class is used.
* access (optional - defaults to property): the strategy Hibernate uses for accessing the property value.
* formula (optional): almost all one-to-one associations map to the primary key of the owning entity. If this is not the case, you can specify another column, columns or expression to join on using an SQL formula. See org.hibernate.test.onetooneformula for an example.
* lazy (optional - defaults to proxy): by default, single point associations are proxied. lazy="no-proxy" specifies that the property should be fetched lazily when the instance variable is first accessed. It requires build-time bytecode instrumentation. lazy="false" specifies that the association will always be eagerly fetched. Note that if constrained="false", proxying is impossible and Hibernate will eagerly fetch the association.

## N + 1 Problem[1]

* The "N" stands for the number of queries that we do not need.
* The "1" stands for the number of queries that we need.

### Problem
* By default, single point associations(@OneToOne and @ManyToOne) are eagerly fetched.
* By default, collection associations(@OneToMany and @ManyToMany) are lazily fetched.
* If we try to select the data which has a single point association with other entities, 
the actual numbers of queries might be 1 + N.  
	
### Solution
* Set the fetched strategy of single point associations to lazy.
    * This could solve the problem completely, If the java code will access the association entities, the N + 1 problem still could happen.
* Depend on the requirement, use join fetch.

## Hibernate SessionFactory 
### Session
* Hibernate Session objects are not thread safe.

### Hibernate SessionFactory getCurrentSession
* Hibernate SessionFactory getCurrentSession() method returns the session bound to the context. 
* We need to configure it in hibernate configuration file like below.
	```
	<property name="hibernate.current_session_context_class">thread</property>
	```
* Or if you want to cooperate with SpringSessionContext
	```
	<prop key="hibernate.current_session_context_class">org.springframework.orm.hibernate5.SpringSessionContext</prop>
	```
* Since this session object belongs to the hibernate context, we don’t need to close it. Once the session factory is closed, this session object gets closed.

### Hibernate SessionFactory openSession
* Hibernate SessionFactory openSession() method always opens a new session. 
* We should close this session object once we are done with all the database operations.

## First Level(Session Level) Cache[2]

* First level cache is associated with “session” object and other session objects in application can not see it.
* The scope of cache objects is of session. Once session is closed, cached objects are gone forever.
* First level cache is enabled by default and you can not disable it.
* When we query an entity first time, it is retrieved from database and stored in first level cache associated with hibernate session.
* The session maintains a Map contains data related to the current Session. When you load data through the primary key, the Session will first check whether there is already in the Map according to the type and the primary key. The data will be returned if there is any and no sql query will be executed.
* The loaded entity can be removed from session using evict() method. The next loading of this entity will again make a database call if it has been removed using evict() method.
* The whole session cache can be removed using clear() method. It will remove all the entities stored in cache.

## Query Cache[4]
* Caching introduces overhead in the area of transactional processing. If you cache results of a query against an object, Hibernate needs to keep track of whether any changes have been committed against the object, and invalidate the cache accordingly. 
* The benefit from caching query results is limited, and highly dependent on the usage patterns of your application. 
* For these reasons, Hibernate disables the query cache by default.

* The query cache does not cache the state of the actual entities in the cache. 
* It caches identifier values and results of value type. 
* Therefore, always use the query cache in conjunction with the second-level cache for those entities which should be cached as part of a query result cache.

### Enabling the query cache
* Set the hibernate.cache.use_query_cache property to true.
    * This setting creates two new cache regions:
        * org.hibernate.cache.internal.StandardQueryCache holds the cached query results.
        * org.hibernate.cache.spi.UpdateTimestampsCache holds timestamps of the most recent updates to queryable tables. These timestamps validate results served from the query cache.
* Adjust the cache timeout of the underlying cache region
    * Set the cache timeout of the underlying cache region for the UpdateTimestampsCache to a higher value than the timeouts of any of the query caches. 
    * It is recommended, to set the UpdateTimestampsCache region never to expire. To be specific, a LRU (Least Recently Used) cache expiry policy is never appropriate.
* Enable results caching for specific queries
    * You need to enable caching for individual queries with org.hibernate.Query.setCacheable(true).
    * This call allows the query to look for existing cache results or add its results to the cache when it is executed.

### Query cache regions
* For fine-grained control over query cache expiration policies, specify a named cache region for a particular query by calling Query.setCacheRegion().
* To force the query cache to refresh one of its regions and disregard any cached results in the region, call org.hibernate.Query.setCacheMode(CacheMode.REFRESH).
* In conjunction with the region defined for the given query, Hibernate selectively refreshes the results cached in that particular region. This is much more efficient than bulk eviction of the region via org.hibernate.SessionFactory.evictQueries().

## Second Level(SessionFactory Level) Cache[3]

* Whenever hibernate session try to load an entity, the very first place it look for cached copy of entity in first level cache (associated with particular hibernate session).
* If cached copy of entity is present in first level cache, it is returned as result of load method.
* If there is no cached entity in first level cache, then second level cache is looked up for cached entity.
* If second level cache has cached entity, it is returned as result of load method. But, before returning the entity, it is stored in first level cache also so that next invocation to load method for entity will return the entity from first level cache itself, and there will not be need to go to second level cache again.
* If entity is not found in first level cache and second level cache also, then database query is executed and entity is stored in both cache levels, before returning as response of load() method.
* Second level cache validate itself for modified entities, if modification has been done through hibernate session APIs.
* If some user or process make changes directly in database, the there is no way that second level cache update itself until “timeToLiveSeconds” duration has passed for that cache region. In this case, it is good idea to invalidate whole cache and let hibernate build its cache once again. You can use below code snippet to invalidate whole hibernate second level cache.

## How to fetch multiple entities by id with Hibernate 5
### How to access the Hibernate Session from JPA
You can call the unwrap() method of the EntityManger to get a Hibernate Session.
```
Session session = em.unwrap(Session.class);
```
### Load multiple entities by their primary key
* Get a typed instance of the MultiIdentifierLoadAccess interface by calling the byMultipleIds(Class entityClass) method on the Hibernate Session.
* Then called the multiLoad(K… ids) method. Hibernate creates one query for this method call and provides the multiple primary keys as parameters to an IN statement.
### Load entities in multiple batches
* There are different reasons to apply batching to these kinds of queries:
    * Not all databases allow an unlimited number of parameters in IN statements.
    * You might detect in your business logic that you don’t need all of them.
    * You might want to remove a batch of entities from the 1st level cache before you fetch the next one.

* Hibernate batch size configuration
    * By default, Hibernate uses the batch size defined in the database specific dialect you use in your application. 
        * Therefore, we don’t need to worry about database limitations. Hibernate’s default behaviour already takes care of it and it’s most often also good enough for performance critical use cases.
    * But for thoes use cases in which you want to change the batch size. You can do this with the withBatchSize(int batchSize) method on the MultiIdentifierLoadAccess interface.
        * Hibernate creates multiple select statements, if the number of provided primary keys exceeds the defined batchSize.
### Don’t fetch entities already stored in 1st level cache
* If you use a JPQL query to fetch a list of entities, Hibernate fetches all of them from the database and checks afterwards if they are already managed in the current session and stored in the 1st level cache. 
* With the new MultiIdentifierLoadAccess interface, you can decide if Hibernate shall check the 1st level cache before it executes the database query. (It is deactivated by default, It can be activated by calling the enableSessionCheck(boolean enabled) method.

## References 

* [1][Lesson 31 - N + 1 Selects Problem](https://youtu.be/KuCqVD9rDqA)
* [2][Understanding Hibernate First Level Cache with Example](https://howtodoinjava.com/hibernate/understanding-hibernate-first-level-cache-with-example/)
* [3][How Hibernate Second Level Cache Works?](https://howtodoinjava.com/hibernate/how-hibernate-second-level-cache-works/)
* [4][Hibernate Community Documentation - Chapter 6. Caching](https://docs.jboss.org/hibernate/orm/4.0/devguide/en-US/html/ch06.html)
* [5][Hibernate SessionFactory](https://www.journaldev.com/3522/hibernate-sessionfactory)
* [6][How to fetch multiple entities by id with Hibernate 5 By Thorben Janssen](https://thorben-janssen.com/fetch-multiple-entities-id-hibernate/)


