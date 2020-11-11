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

## Transactions and Concurrency
* Through Session, which is also a transaction-scoped cache, Hibernate provides repeatable reads for lookup by identifier and entity queries and not reporting queries that return scalar values.

### Session and transaction scopes
* SessionFactory: A SessionFactory is an expensive-to-create, threadsafe object, intended to be shared by all application threads. It is created once, usually on application startup, from a Configuration instance.
* Session: A Session is an inexpensive, non-threadsafe object that should be used once and then discarded for: a single request, a conversation or a single unit of work. A Session will not obtain a JDBC Connection, or a Datasource, unless it is needed. It will not consume any resources until used.
* Database transaction: In order to reduce lock contention in the database, a database transaction has to be as short as possible. Long database transactions will prevent your application from scaling to a highly concurrent load.

* What is the scope of a unit of work? 
* Can a single Hibernate Session span several database transactions, or is this a one-to-one relationship of scopes? 
* When should you open and close a Session and how do you demarcate the database transaction boundaries? 

* Do not use the session-per-operation antipattern: 
	* Do not open and close a Session for every simple database call in a single thread. 
	* The same is true for database transactions. Database calls in an application are made using a planned sequence; they are grouped into atomic units of work. 
	* This also means that auto-commit after every single SQL statement is useless in an application as this mode is intended for ad-hoc SQL console work. 
	* Hibernate disables, or expects the application server to disable, auto-commit mode immediately. 
	* Database transactions are never optional. All communication with a database has to occur inside a transaction. Auto-commit behavior for reading data should be avoided, as many small transactions are unlikely to perform better than one clearly defined unit of work.

* The most common pattern: session-per-request
	* A new Hibernate Session is opened, and all database operations are executed in this unit of work. 
	* On completion of the work, and once the response for the client has been prepared, the session is flushed and closed.
	* Use a single database transaction to serve the clients request, starting and committing it when you open and close the Session. The relationship between the two is one-to-one and this model is a perfect fit for many applications.
	* Hibernate provides built-in management of the "current session" to simplify this pattern. 
		* Start a transaction when a server request has to be processed, and end the transaction before the response is sent to the client.
		* Your application code can access a "current session" to process the request by calling sessionFactory.getCurrentSession(). You will always get a Session scoped to the current database transaction.
		* You can extend the scope of a Session and database transaction until the "view has been rendered". (especially useful in servlet applications)

### Long conversations

### Considering object identity

* There are two different notions of identity
	* Database Identity: foo.getId().equals( bar.getId() )
	* JVM Identity: foo==bar

* Therefore, An application can concurrently access the same persistent state in two different Sessions. But an instance of a persistent class is never shared between two Session instances. 

* Within a Session
	* The application does not need to synchronize on any business object, as long as it maintains a single thread per Session. Within a Session the application can safely use == to compare objects.

* Outside of a Session
	* The developer has to override the equals() and hashCode() methods in persistent classes and implement their own notion of object equality. 
		* There is one caveat: never use the database identifier to implement equality. Use a business key that is a combination of unique, usually immutable, attributes. 

### Common issues
* A Session is not thread-safe. Things that work concurrently, like HTTP requests, session beans, or Swing workers, will cause race conditions if a Session instance is shared.

* An exception thrown by Hibernate means you have to rollback your database transaction and close the Session immediately.
	* Rolling back the database transaction does not put your business objects back into the state they were at the start of the transaction. This means that the database state and the business objects will be out of sync. 
	* Usually this is not a problem, because exceptions are not recoverable and you will have to start over after rollback anyway.

* The Session caches every object that is in a persistent state (watched and checked for dirty state by Hibernate). If you keep it open for a long time or simply load too much data, it will grow endlessly until you get an OutOfMemoryException. 
	* One solution is to call clear() and evict() to manage the Session cache, but you should consider a Stored Procedure if you need mass data operations. 


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

## Improving performance

### Fetching strategies
* We have two orthogonal notions here: `when` is the association fetched and `how` is it fetched.
* Fetch strategies can be declared in the O/R mapping metadata, or over-ridden by a particular HQL or Criteria query.
* Hibernate defines the following fetching strategies
    * Join fetching: Hibernate retrieves the associated instance or collection in the same SELECT, using an OUTER JOIN.
    * Select fetching: a second SELECT is used to retrieve the associated entity or collection. Unless you explicitly disable lazy fetching by specifying lazy="false", this second select will only be executed when you access the association.
    * Subselect fetching: a second SELECT is used to retrieve the associated collections for all entities retrieved in a previous query or fetch. Unless you explicitly disable lazy fetching by specifying lazy="false", this second select will only be executed when you access the association.
    * Batch fetching: an optimization strategy for select fetching. Hibernate retrieves a batch of entity instances or collections in a single SELECT by specifying a list of primary or foreign keys.

* Hibernate also distinguishes between
    * Immediate fetching: an association, collection or attribute is fetched immediately when the owner is loaded.
    * Lazy collection fetching: a collection is fetched when the application invokes an operation upon that collection. This is the default for collections.
    * "Extra-lazy" collection fetching: individual elements of the collection are accessed from the database as needed. Hibernate tries not to fetch the whole collection into memory unless absolutely needed. It is suitable for large collections.
    * Proxy fetching: a single-valued association is fetched when a method other than the identifier getter is invoked upon the associated object.
    * "No-proxy" fetching: a single-valued association is fetched when the instance variable is accessed. Compared to proxy fetching, this approach is less lazy; the association is fetched even when only the identifier is accessed. It is also more transparent, since no proxy is visible to the application. This approach requires buildtime bytecode instrumentation and is rarely necessary.
    * Lazy attribute fetching: an attribute or single valued association is fetched when the instance variable is accessed. This approach requires buildtime bytecode instrumentation and is rarely necessary.

### Working with lazy associations
* By default, Hibernate uses lazy select fetching for collections and lazy proxy fetching for single-valued associations.
* Hibernate does not support lazy initialization for detached objects. Access to a lazy association outside of the context of an open Hibernate session will result in an exception.

### Tuning fetch strategies
* Select fetching (the default) is extremely vulnerable to N+1 selects problems, so we might want to enable join fetching in the mapping document.
	* The fetch strategy defined in the mapping document affects:
		* retrieval via get() or load()
		* retrieval that happens implicitly when an association is navigated
		* Criteria queries
		* HQL queries if subselect fetching is used
* The mapping document is not used to customize fetching. Instead, we keep the default behavior, and override it for a particular transaction.
	* Using left join fetch in HQL. This tells Hibernate to fetch the association eagerly in the first select, using an outer join. 
	* In the Criteria query API, you would use setFetchMode(FetchMode.JOIN).

* A completely different approach to problems with N+1 selects is to use the second-level cache.

### Single-ended association proxies
* At startup, Hibernate generates proxies by default for all persistent classes and uses them to enable lazy fetching of many-to-one and one-to-one associations.
* The mapping file may declare an interface to use as the proxy interface for that class, with the proxy attribute. 
* By default, Hibernate uses a subclass of the class. The proxied class must implement a default constructor with at least package visibility. This constructor is recommended for all persistent classes.
* TBD...

### Best Practices
* TBD...

## Hibernate-mapping v.s. JAP Annotations
* Different OneToOne mappings
    * [Hibernate one-to-one](https://docs.jboss.org/hibernate/core/3.3/reference/en/html/mapping.html#mapping-declaration-onetoone)
    * [@OneToOne](https://docs.oracle.com/javaee/6/api/javax/persistence/OneToOne.html)
* TBD... 

## References 

* [1][Lesson 31 - N + 1 Selects Problem](https://youtu.be/KuCqVD9rDqA)
* [2][Understanding Hibernate First Level Cache with Example](https://howtodoinjava.com/hibernate/understanding-hibernate-first-level-cache-with-example/)
* [3][How Hibernate Second Level Cache Works?](https://howtodoinjava.com/hibernate/how-hibernate-second-level-cache-works/)
* [4][Hibernate Community Documentation - Chapter 6. Caching](https://docs.jboss.org/hibernate/orm/4.0/devguide/en-US/html/ch06.html)
* [5][Hibernate SessionFactory](https://www.journaldev.com/3522/hibernate-sessionfactory)
* [6][How to fetch multiple entities by id with Hibernate 5 By Thorben Janssen](https://thorben-janssen.com/fetch-multiple-entities-id-hibernate/)


