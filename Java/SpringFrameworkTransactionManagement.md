# Spring Framework Transaction Management
## Understanding the Spring Framework transaction abstraction
* Transaction strategy
    * Defined by the org.springframework.transaction.PlatformTransactionManager interface
* TransactionStatus
    * TransactionStatus might represent a new transaction, or can represent an existing transaction if a matching transaction exists in the current call stack.
* TransactionDefinition
    * Isolation, Propagation, Timeout, Read-only status
* HibernateTransactionManager
    * It needs a reference to the SessionFactory

## Understanding the Spring Framework’s declarative transaction implementation
* The most important concepts to grasp with regard to the Spring Framework’s declarative transaction support are that this support is enabled via AOP proxies, and that the transactional advice is driven by metadata (currently XML- or annotation-based). 
* The combination of AOP with transactional metadata yields an AOP proxy that uses a TransactionInterceptor in conjunction with an appropriate PlatformTransactionManager implementation to drive transactions around method invocations.

### Using @Transactional
* The <tx:annotation-driven/> element switches on the transactional behavior.
* Spring recommends that you only annotate concrete classes (and methods of concrete classes) with the @Transactional annotation. (proxy-target-class="true")
* The @Transactional annotation placing on an interface (or an interface method) only works if you are using interface-based proxies. (proxy-target-class="false")

* In proxy mode (which is the default), only external method calls coming in through the proxy are intercepted. 
    * This means that self-invocation, in effect, a method within the target object calling another method of the target object, will not lead to an actual transaction at runtime even if the invoked method is marked with @Transactional. 
    * The proxy must be fully initialized to provide the expected behaviour so you should not rely on this feature in your initialization code, i.e. @PostConstruct.

* The alternative mode "aspectj" instead weaves the affected classes with Spring’s AspectJ transaction aspect, modifying the target class byte code to apply to any kind of method call. AspectJ weaving requires spring-aspects.jar in the classpath as well as load-time weaving (or compile-time weaving) enabled.
    
### Transaction propagation
* Required
    * A logical transaction scope is created for each method upon which the setting is applied. 
    * Each such logical transaction scope can determine rollback-only status individually, with an outer transaction scope being logically independent from the inner transaction scope.
    * All these scopes will be mapped to the same physical transaction. So a rollback-only marker set in the inner transaction scope does affect the outer transaction’s chance to actually commit.
    * (!)In the case where an inner transaction scope sets the rollback-only marker:
        * If an inner transaction (of which the outer caller is not aware) silently marks a transaction as rollback-only, the outer caller still calls commit. 
        * The outer caller needs to receive an UnexpectedRollbackException to indicate clearly that a rollback was performed instead.
         
* RequiresNew
    * A completely independent transaction for each affected transaction scope is created.
    * The underlying physical transactions are different and hence can commit or roll back independently.
    * An outer transaction not affected by an inner transaction’s rollback status.
        
* Nested
    * It uses a single physical transaction with multiple savepoints that it can roll back to.
    * Such partial rollbacks allow an inner transaction scope to trigger a rollback for its scope, with the outer transaction being able to continue the physical transaction despite some operations having been rolled back. 
    * This setting is typically mapped onto JDBC savepoints, so will only work with JDBC resource transactions. (DataSourceTransactionManager)

## References
* [16. Transaction Management](https://docs.spring.io/spring-framework/docs/4.2.x/spring-framework-reference/html/transaction.html)
* [Spring Transaction Propagation in a Nutshell](https://dzone.com/articles/spring-transaction-propagation)


