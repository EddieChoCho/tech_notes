### Keywords
* OFFSET: like page, start with 0 index
* UNION v.s. UNION ALL
    * UNION command combines only distinct values of the result set of multiple SELECT statements. 
    * UNION ALL command combines the result set of multiple SELECT statements (allows duplicate values, which has better performance).

### Consideration for null case
* Soltuion-1: Take the result as a temp table
```
SELECT (SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT 1 OFFSET 1) 
AS SecondHighestSalary;
```
* Soltuion-2: Use IFNULL funtion
```
SELECT IFNULL( 
    (SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT 1 OFFSET 1),
    NULL) AS SecondHighestSalary
```

### MySQL Query Optimization Best Practices

* Use EXPLAIN keyword to see the execution plan for the query
	* Check the index usage
	* Check rows scanned
* Use LIMIT 1 clause when retrieving Unique Row
	* Helps aggregate functions like MIN or MAX
* Try to Convert <> operator to = operator
	* = operator increases chances of index usage

* Avoid using SELECT *
	* Forces full table scan
	* Wastes network bandwidth

* Split big Delete Update, or INSERT query into multiple smaller queries
* Use appropriate data types for columns
	* Smaller columns are faster for performance
* MySQl query cache is case and space sensitive
	* Use same query case for repeat queries
* Index columns in the WHERE clause
* Index columns used in JOIN
* Use UNION ALL instead of UNION if duplicate data is permissiable in resultset
* Table order does not matter when INNER JOIN clause is used
* If column used in ORDER BY clause are indexed they help with performance
* Use LIMIT clause to implement pagination logic

### Myth Directory

* [Indexes Can Degenerate](https://use-the-index-luke.com/sql/myth-directory/indexes-can-degenerate)
* [Most Selective First](https://use-the-index-luke.com/sql/myth-directory/most-selective-first)
* [Oracle Cannot Index NULL](https://use-the-index-luke.com/sql/myth-directory/null-cannot-be-indexed)
* [Dynamic SQL is Slow](https://use-the-index-luke.com/sql/myth-directory/dynamic-sql-is-slow)
* [Select * is Bad](https://use-the-index-luke.com/blog/2013-08/its-not-about-the-star-stupid)

### References

* [MySQL Query Optimization Best Practices](https://youtu.be/cYnT_-jy_Y8)
* [We need tool support for keyset pagination](https://use-the-index-luke.com/no-offset)
