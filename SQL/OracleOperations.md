## Latency and Bandwidth.
* Bandwidth has only a minor impact on the response time but latencies have a huge impact. 
* That means that the number of database round trips is more important for the response time than the amount of data transferred.

## Index and Table Access
* INDEX UNIQUE SCAN
    * The INDEX UNIQUE SCAN performs the B-tree traversal only. 
    * The database uses this operation if a unique constraint ensures that the search criteria will match no more than one entry.

* INDEX RANGE SCAN
    * The INDEX RANGE SCAN performs the B-tree traversal and follows the leaf node chain to find all matching entries. 
    * The so-called index filter predicates often cause performance problems for an INDEX RANGE SCAN. 
        * TDB: explains how to identify them.

* INDEX FULL SCAN
    * Reads the entire index—all rows—in index order.
    * Depending on various system statistics, the database might perform this operation if it needs all rows in index order—e.g., because of a corresponding order by clause. Instead, the optimizer might also use an INDEX FAST FULL SCAN and perform an additional sort operation. See Chapter 6, “Sorting and Grouping”.

* INDEX FAST FULL SCAN
    * Reads the entire index—all rows—as stored on the disk. 
    * This operation is typically performed instead of a full table scan if all required columns are available in the index. 
    * Similar to TABLE ACCESS FULL, the INDEX FAST FULL SCAN can benefit from multi-block read operations. 
    * TBD: See Chapter 5, “Clustering Data”.

* TABLE ACCESS BY INDEX ROWID
    * Retrieves a row from the table using the ROWID retrieved from the preceding index lookup. 
    * TBD: See also Chapter 1, “Anatomy of an SQL Index”.

* TABLE ACCESS FULL
    * This is also known as full table scan. Reads the entire table—all rows and columns—as stored on the disk. 
    * Although multi-block read operations improve the speed of a full table scan considerably, it is still one of the most expensive operations. 
    * Besides high IO rates, a full table scan must inspect all table rows so it can also consume a considerable amount of CPU time. 
    * TBD: See also “Full Table Scan”.

## Joins
Generally join operations process only two tables at a time. In case a query has more joins, they are executed sequentially: first two tables, then the intermediate result with the next table. In the context of joins, the term “table” could therefore also mean “intermediate result”.

### Nested LOOPS JOIN
* It works like using two nested queries: the outer or driving query to fetch the results from one table and a second query for each row from the driving query to fetch the corresponding data from the other table.
* like N+1 selects problem cause by ORM tool
* The nested loops join delivers good performance if the driving query returns a small result set. 
* Otherwise, the optimizer might choose an entirely different join algorithm—like the hash join described in the next section, but this is only possible if the application uses a join to tell the database what data it actually needs.


### HASH JOIN
* It loads the candidate records from one side of the join into a hash table that can be probed very quickly for each row from the other side of the join. 
* Hash joins cannot perform joins that have range conditions in the join predicates (theta joins).

#### Tuning a hash join
Requires an entirely different indexing approach than the nested loops join. 
##### The indexing strategy for a hash join
* There is no need to index the join columns - Hash joins do not need indexes on the join predicates. They use the hash table instead.
* Only indexes for independent where predicates improve hash join performance.-A hash join uses indexes only if the index supports the independent predicates.

#### Other tips
* Select fewer columns to improve hash join performance.
    * Which is a challenge for most ORM tools.
* Reduce the hash table size to improve performance; either horizontally (less rows) or vertically (less columns).

### Sort Merge
* The sort-merge join combines two sorted lists like a zipper. Both sides of the join must be sorted by the join predicates.

#### Parts like hash join 
* Need an index for the independent conditions to read all candidate records in one shot. 
* Indexing the join predicates is useless. 

### Parts different from hash join 
* The join order does not make any difference—not even for performance. This property is very useful for outer joins. 


## Sorting and Grouping
SORT ORDER BY
Sorts the result according to the order by clause. This operation needs large amounts of memory to materialize the intermediate result (not pipelined). See also “Indexing Order By”.

SORT ORDER BY STOPKEY
Sorts a subset of the result according to the order by clause. Used for top-N queries if pipelined execution is not possible. See also “Querying Top-N Rows”.

SORT GROUP BY
Sorts the result set on the group by columns and aggregates the sorted result in a second step. This operation needs large amounts of memory to materialize the intermediate result set (not pipelined). See also “Indexing Group By”.

SORT GROUP BY NOSORT
Aggregates a presorted set according the group by clause. This operation does not buffer the intermediate result: it is executed in a pipelined manner. See also “Indexing Group By”.

HASH GROUP BY
Groups the result using a hash table. This operation needs large amounts of memory to materialize the intermediate result set (not pipelined). The output is not ordered in any meaningful way. See also “Indexing Group By”.

## Top-N Queries
The efficiency of top-N queries depends on the execution mode of the underlying operations. They are very inefficient when aborting non-pipelined operations such as SORT ORDER BY.

COUNT STOPKEY
Aborts the underlying operations when the desired number of rows was fetched. See also “Querying Top-N Rows”.

WINDOW NOSORT STOPKEY
Uses a window function (over clause) to abort the execution when the desired number of rows was fetched. See also “Using Window Functions for Efficient Pagination”.

## References
* [Explain the Explain by Maria Colgan](https://github.com/EddieChoCho/tech-talks-note/blob/master/2020/ExplainTheExplain.md)
* [Use The Index, Luke!](https://use-the-index-luke.com/)