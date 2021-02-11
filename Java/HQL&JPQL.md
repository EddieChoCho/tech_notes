## HQL and JPQL
* The Hibernate Query Language (HQL) and Java Persistence Query Language (JPQL) are both object model focused query languages similar in nature to SQL. 
* JPQL is a heavily-inspired-by subset of HQL. A JPQL query is always a valid HQL query, the reverse is not true, however.
* JPA Query API
    * Query#getResultList() - executes the select query and returns back the list of results.
    * Query#getResultStream() - executes the select query and returns back a Stream over the results.
    * Query#getSingleResult() - executes the select query and returns a single result. If there were more than one result an exception is thrown.
    
* Hibernate Query API
    * Query#list - executes the select query and returns back the list of results.
    * Query#uniqueResult - executes the select query and returns the single result. If there were more than one result an exception is thrown.

### Query scrolling

#### Query#scroll
* Hibernate offers additional, specialized methods for scrolling the query and handling results using a server-side cursor.
* Query#scroll works in tandem with the JDBC notion of a scrollable ResultSet.
* The Query#scroll method is overloaded:
    * The main form accepts a single argument of type org.hibernate.ScrollMode which indicates the type of scrolling to be used.
    * The second form takes no argument and will use the ScrollMode indicated by Dialect#defaultScrollMode.
* Query#scroll returns a org.hibernate.ScrollableResults which wraps the underlying JDBC (scrollable) ResultSet and provides access to the results. 
* Unlike a typical forward-only ResultSet, the ScrollableResults allows you to navigate the ResultSet in any direction.
```java
try ( ScrollableResults scrollableResults = session.createQuery(
		"select p " +
		"from Person p " +
		"where p.name like :name" )
		.setParameter( "name", "J%" )
		.scroll()
) {
	while(scrollableResults.next()) {
		Person person = (Person) scrollableResults.get()[0];
		process(person);
	}
}
```
* It is good practice to close the ScrollableResults explicitly.
    * Since this form holds the JDBC ResultSet open, the application should indicate when it is done with the ScrollableResults by calling its close() method (as inherited from java.io.Closeable so that ScrollableResults will work with try-with-resources blocks).
    * If left unclosed by the application, Hibernate will automatically close the underlying resources (e.g. ResultSet and PreparedStatement) used internally by the ScrollableResults when the current transaction is ended (either commit or rollback).

#### Query#iterate
* Hibernate also supports Query#iterate, which is intended for loading entities when it is known that the loaded entries are already stored in the second-level cache.
    * The idea behind iterate is that just the matching identifiers will be obtained in the SQL query. 
    * From these the identifiers are resolved by second-level cache lookup. 
    * If these second-level cache lookups fail, additional queries will need to be issued against the database.
    
* This operation can perform significantly better for loading large numbers of entities that for certain already exist in the second-level cache. 
* In cases where many of the entities do not exist in the second-level cache, this operation will almost definitely perform worse.

#### Query#stream
* Internally, the stream() behaves like a Query#scroll and the underlying result is backed by a ScrollableResults.
* Just like with ScrollableResults, you should always close a Hibernate Stream either explicitly or using a try-with-resources block.

### Query streaming
* Since version 2.2, the JPA Query interface offers support for returning a Stream via the getResultStream method.
* Just like the scroll method, you can use a try-with-resources block to close the Stream prior to closing the currently running Persistence Context.
* Since Hibernate 5.4, the Stream is also closed when calling a terminal operation, as illustrated by the following example.

## Select statements
* The BNF(Backus–Naur form) for SELECT statements in HQL:
    ```
    select_statement :: =
        [select_clause]
        from_clause
        [where_clause]
        [groupby_clause]
        [having_clause]
        [orderby_clause]
    ```
* The select statement in JPQL is exactly the same as for HQL except that JPQL requires a select_clause, whereas HQL does not.
* Even though HQL does not require the presence of a select_clause, it is generally good practice to include one. For simple queries the intent is clear and so the intended result of the select_clause is easy to infer. But on more complex queries that is not always the case.

## Update statements
* The BNF for UPDATE statements is the same in HQL and JPQ:
    ```
    update_statement ::=
        update_clause [where_clause]

    update_clause ::=
        UPDATE entity_name [[AS] identification_variable]
        SET update_item {, update_item}*

    update_item ::=
        [identification_variable.]{state_field | single_valued_object_field} = new_value

    new_value ::=
        scalar_expression | simple_entity_expression | NULL
    ```
* An UPDATE statement is executed using the executeUpdate() of either org.hibernate.query.Query or javax.persistence.Query. 
* The int value returned by the executeUpdate() method indicates the number of entities affected by the operation. This may or may not correlate to the number of rows affected in the database.
* Neither UPDATE nor DELETE statements allow implicit joins. Their form already disallows explicit joins too.

## Delete statements
* The BNF for DELETE statements is the same in HQL and JPQL:
    ```
    delete_statement ::=
        delete_clause [where_clause]
    
    delete_clause ::=
        DELETE FROM entity_name [[AS] identification_variable]
    ``` 
* A DELETE statement is also executed using the executeUpdate() method.

## Insert statements
* There is no JPQL equivalent to HQL-style INSERT statements.
* The BNF for an HQL INSERT statement:
    ```
    insert_statement ::=
        insert_clause select_statement
    
    insert_clause ::=
        INSERT INTO entity_name (attribute_list)
    
    attribute_list ::=
        state_field[, state_field ]*
    ```    
* TBD...

## Explicit joins
* The FROM clause can also contain explicit relationship joins using the join keyword. These joins can be either `inner` or `left outer` style joins.
* HQL also defines a `WITH` clause to qualify the join conditions.
* The HQL-style `WITH` keyword is specific to Hibernate. JPQL defines the `ON` clause for this feature.

### Fetch join
* An important use case for explicit joins is to define FETCH JOINs which override the laziness of the joined association.
* Fetch joins are not valid in sub-queries.
    * Care should be taken when fetch joining a collection-valued association which is in any way further restricted (the fetched collection will be restricted too). For this reason, it is usually considered best practice not to assign an identification variable to fetched joins except for the purpose of specifying nested fetch joins.
    * Fetch joins should not be used in paged queries (e.g. setFirstResult() or setMaxResults()), nor should they be used with the scroll() or iterate() features.

## References 
* [Hibernate User Guide](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html)