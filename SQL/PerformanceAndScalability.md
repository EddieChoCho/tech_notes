## Performance and Scalability
### Performance Impacts of Data Volume
* The amount of data stored in a database has a great impact on its performance. It is usually accepted that a query becomes slower with additional data in the database. 
* The key questions when discussing database scalability:
	* How great is the performance impact if the data volume doubles? 
	* How can we improve this ratio? 

* Scalability shows the dependency of performance on factors like the.
* A performance value is just a single data point on a scalability chart.

* Check the cost value of the execution plans
	* Even though the cost values reflect the speed difference, the reason is not visible in the execution plan.
	* Two ingredients that make an index lookup slow: (1) the table access, and (2) scanning a wide index range.
* Pay attention to the predicate information. An execution plan without predicate information is incomplete. 
* The query which only use an access predicate has better performance than the query use an access predicate and a filter predicate.
	* Filter predicate
		* Filter predicates are like unexploded ordnance devices. They can explode at any time.
		* INDEX RANGE SCAN + filter predicates = The index has at least three columns, two of them(the first and the thrid columns) have been used as the query paramters.
	* Access predicate
		* The index has at least two columns, two of them(the first and the second columns) have been used as the query paramters. 
		* We can nonetheless not say anything about the order of the columns.

### Performance Impacts of System Load		
* The optimizer is not using an index because it is the “right” one for the query, rather because it is more efficient than a full table scan. 
* That does not mean it is the optimal index for the query.

## Predicate Information
* TBD...

## Reference
* [Performance and Scalability by Markus Winand](https://use-the-index-luke.com/sql/testing-scalability)