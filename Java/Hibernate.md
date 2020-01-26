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

## First-Level Cache[2]

* First level cache is associated with “session” object and other session objects in application can not see it.
* The scope of cache objects is of session. Once session is closed, cached objects are gone forever.
* First level cache is enabled by default and you can not disable it.
* When we query an entity first time, it is retrieved from database and stored in first level cache associated with hibernate session.
* If we query same object again with same session object, it will be loaded from cache and no sql query will be executed.
* The loaded entity can be removed from session using evict() method. The next loading of this entity will again make a database call if it has been removed using evict() method.
* The whole session cache can be removed using clear() method. It will remove all the entities stored in cache.


## Second Level Cache[3]

* Whenever hibernate session try to load an entity, the very first place it look for cached copy of entity in first level cache (associated with particular hibernate session).
* If cached copy of entity is present in first level cache, it is returned as result of load method.
* If there is no cached entity in first level cache, then second level cache is looked up for cached entity.
* If second level cache has cached entity, it is returned as result of load method. But, before returning the entity, it is stored in first level cache also so that next invocation to load method for entity will return the entity from first level cache itself, and there will not be need to go to second level cache again.
* If entity is not found in first level cache and second level cache also, then database query is executed and entity is stored in both cache levels, before returning as response of load() method.
* Second level cache validate itself for modified entities, if modification has been done through hibernate session APIs.
* If some user or process make changes directly in database, the there is no way that second level cache update itself until “timeToLiveSeconds” duration has passed for that cache region. In this case, it is good idea to invalidate whole cache and let hibernate build its cache once again. You can use below code snippet to invalidate whole hibernate second level cache.


## References 

* [1][Lesson 31 - N + 1 Selects Problem](https://youtu.be/KuCqVD9rDqA)
* [2][Understanding Hibernate First Level Cache with Example](https://howtodoinjava.com/hibernate/understanding-hibernate-first-level-cache-with-example/)
* [3][How Hibernate Second Level Cache Works?](https://howtodoinjava.com/hibernate/how-hibernate-second-level-cache-works/)


