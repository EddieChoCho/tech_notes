
## Pessimistic lock
We can think of two concurrency control mechanisms which allow us to do that: 
1. Setting the proper transaction isolation level 
	* The isolation level is set once the connection is created and it affects every statement within that connection.
2. Setting a lock on data that we need at the moment.
	* We can use pessimistic locking which uses database mechanisms for reserving more granular exclusive access to the data.
	* Pessimistic lock could ensure that no other transactions can modify or delete reserved data.

### There are two types of locks we can retain: 
* Exclusive lock 
	* In order to modify or delete the reserved data, we need to have an exclusive lock.
	* We can acquire exclusive locks using ‘SELECT … FOR UPDATE‘ statements.
* Shared lock
	* We could read but not write in data when someone else holds a shared lock

### Lock Modes
* PESSIMISTIC_READ
	* Whenever we want to just read data and don't encounter dirty reads, we could use PESSIMISTIC_READ (shared lock). We won't be able to make any updates or deletes though.
	
* PESSIMISTIC_WRITE
	* Any transaction that needs to acquire a lock on data and make changes to it should obtain the PESSIMISTIC_WRITE lock. According to the JPA specification, holding PESSIMISTIC_WRITE lock will prevent other transactions from reading, updating or deleting the data.
	* Please note that some database systems implement [multi-version concurrency control](https://en.wikipedia.org/wiki/Multiversion_concurrency_control) 
	which allows readers to fetch data that has been already blocked.

* PESSIMISTIC_FORCE_INCREMENT
    * This lock works similarly to PESSIMISTIC_WRITE, but it was introduced to cooperate with versioned entities – entities which have an attribute annotated with @Version.
    * Any updates of versioned entities could be preceded with obtaining the PESSIMISTIC_FORCE_INCREMENT lock. Acquiring that lock results in updating the version column.
    
### Using Pessimistic Lock
There are a few possible ways to configure a pessimistic lock on a single record or group of records.
* Find
```
entityManager.find(Student.class, studentId, LockModeType.PESSIMISTIC_READ);
```
* Query
```
Query query = entityManager.createQuery("from Student where studentId = :studentId");
query.setParameter("studentId", studentId);
query.setLockMode(LockModeType.PESSIMISTIC_WRITE);
query.getResultList()
```
* Explicit Locking
```
Student resultStudent = entityManager.find(Student.class, studentId);
entityManager.lock(resultStudent, LockModeType.PESSIMISTIC_WRITE);
```
* Refresh
```
Student resultStudent = entityManager.find(Student.class, studentId);
entityManager.refresh(resultStudent, LockModeType.PESSIMISTIC_FORCE_INCREMENT);
```
* NamedQuery
```
@NamedQuery(name="lockStudent", query="SELECT s FROM Student s WHERE s.id LIKE :studentId", lockMode = PESSIMISTIC_READ)
```
    
### Lock Scope
* Lock scope parameter defines how to deal with locking relationships of the locked entity.
* PessimisticLockScope.NORMAL
    * Default scope. With this locking scope, we lock the entity itself. When used with joined inheritance it also locks the ancestors.
* PessimisticLockScope.EXTENDED
    * Same functionality as NORMAL but it's able to block related entities in a join table.
    * Not all persistence providers support lock scopes.
    
### Setting Lock Timeout
* The milliseconds that we want to wait for obtaining a lock until the LockTimeoutException occurs.
* Property ‘javax.persistence.lock.timeout'
* Set value to zero ->  specify ‘no wait' locking. (Some database drivers might not support setting value to zero.)

### Conclusion
* When setting the proper isolation level is not enough to cope with concurrent transactions. 
* The pessimistic locking enables us to isolate and orchestrate different transactions so they don't access the same resource at the same time.
* The behavior of pessimistic locks depends on persistence provider we work with. 
    

## Reference
* [Pessimistic Locking in JPA](https://www.baeldung.com/jpa-pessimistic-locking)
