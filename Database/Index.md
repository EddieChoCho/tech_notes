# Index

## Terminology
* Search key: every index has an associated search key
* Primary index: search key contains primary key
* Secondary index: search key does not contain primary key

## Hash-Based Indexes
* Designed for equality selections
* Uses a hashing function
	* Search values -> bucket numbers
* Bucket
	* Primary page plus zero or more overflow pages
* Based on static or dynamic hashing techniques

### Static Hashing
* The number of bucket N is fixed
* Overflow pages if needed
* h(k) mod N = bucket to which data entry with key k belongs
* Records having the same hash value are stored in the same bucket
* Search Cost of Static Hashing
	* Assume index has B blocks and has N buckets
	* Then each bucket is about B/N blocks long
* Limitation of Static Hashing
	* Seach cost: B/N
	* To increase efficiency, we need to increase the #buckets(N): Best when 1 block per bucket
	* However, a large #buckets leads to wasted space: empty pages waiting the index to grow into it.
	* Hard to decide the number of #buckets.
	* Double the number of #buckets when a bucket is full is not a good idea - Redistributing records(re-hash) is costly

### Extendable Hash Indexes
* Use directory: pointers to buckets
* Double #buckets by doubling the directory, no need for re-hash
* Splitting just the bucket that overflowed
* Global depth of directory: Max #bits needed to tell which bucket an entry belongs to
* Local depth of a bucket: #bits used the determine if an entry belongs to this bucket
* Remarks
	* At most 1 pag split for each insert
	* Cheap doubling
		* When local depth of bucket = global depth
		* Only 3 page access(1 directory page, 2 data pages)
	* Still has overflow page
		* But only when there are a lot of records with same key value

### Is Hash-Based Index Good Enough?
* Hash-based indexes are good for equality selection
* However, cannot support rande searches
* We now consider an index structured as a search tree
	* Speeds up search by storing values
	* Supports both range  and equality selections

## B-Tree Indexes
* Index records are sorted on dataVal in each page
* M-way balanced search tree
	* O(logM(#data-records)) for equality search & update
	* O(#data-records) for range search

### Searching: Finding all index records having a specified dataValue v
* Search begins at root
* Fetches child block pointed by parent util leaf
	* Search cost: O(tree height), usually < 5

### Insertion
* Step1: Search the index with the inserted dataVal
* Step2: Insert the new index record into the target lead block
* What if the block has no more room? Split it
* Splitting
	1. Allocate a new block in the index file
	2. Move the high-valued half of the index record into this new block
	3. Create a directory record for the new block(New directory record has the dataVal of the first value in the new block)
	4. Insert the new directory record into the same level-0 directory block
	5. Recursively split directory block if necessary
	* Update cost: O(tree height)

* Duplicate DataVals
	* When splitting a leaf block, we must place all records with same dataVal in same block
	* What if there are too many records with same dataVal? -> Overflow blocks

* Overflow Blocks
	* Keep records of the same dataVal
	* Chained by primary blocks
	* First dataVal in primary leaf block = dataVal in overflow block

### Deletion
1. Search the index with the target dataVal
2. Delete the index record in a leaf block
3. Move the next records one-slot ahead
4. Merge blocks if #records is less than a threshold
5. Recursive delete on parents

## Slow Indexes, Part I

* Rebuilding an index does not improve performance on the long run.

### Ingredients for a slow index lookup

* Leaf node chain
    * In a case that an index lookup not only needs to perform the tree traversal, but also needs to follow the leaf node chain.
    * e.g. not-unique 
* Accessing the table
    * Even a single leaf node might contain many hits—often hundreds. 
    * The corresponding table data is usually scattered across many table blocks. That means that there is an additional table access for each hit.

### An index lookup requires three steps

* (1) the tree traversal
* (2) following the leaf node chain
* (3) fetching the table data. 
    * The tree traversal is the only step that has an upper bound for the number of accessed blocks—the index depth. 
    * The other two steps might need to access many blocks—they cause a slow index lookup.

### Three distinct operations that describe a basic index lookup of the Oracle database

* INDEX UNIQUE SCAN
    * It performs the tree traversal only. 
    * The Oracle database uses this operation if a unique constraint ensures that the search criteria will match no more than one entry.
    
* INDEX RANGE SCAN
    * It performs the tree traversal and follows the leaf node chain to find all matching entries. 
    * This is the fall­back operation if multiple entries could possibly match the search criteria.
    * The important point is that an INDEX RANGE SCAN can potentially read a large part of an index. If there is one more table access for each row, the query can become slow even when using an index.

* TABLE ACCESS BY INDEX ROWID
    * This operation retrieves the row from the table. 
    * This operation is (often) performed for every matched record from a preceding index scan operation.

# The Where Clause

* This chapter explains how different operators affect index usage and how to make sure that an index is usable for as many queries as possible. 
* The last section shows common anti-patterns and presents alternatives that deliver better performance.

## The Equality Operator
* This section shows how to verify index usage and explains how concatenated indexes can optimize combined conditions. 

### Primary Keys
* The database automatically creates an index for the primary key.
*  Select from pk:

| Id  | Operation                    | Name           | Rows | Cost |
| --- | ---------------------------- | -------------- | ---- | ---- |
|  0  | SELECT STATEMENT             |                |   1  |   2  |
|  1  |  TABLE ACCESS BY INDEX ROWID | EMPLOYEES      |   1  |   2  |
|  2  |   INDEX UNIQUE SCAN          | EMPLOYEES_PK   |   1  |   1  |

* INDEX UNIQUE SCAN
	* The operation that only traverses the index tree. 
	* It fully utilizes the logarithmic scalability of the index to find the entry very quickly—almost independent of the table size.

* TABLE ACCESS BY INDEX ROWID operation. This operation can become a performance bottleneck
	* but there is no such risk in connection with an INDEX UNIQUE SCAN. This operation cannot deliver more than one entry 
	* so it cannot trigger more than one table access. 
	* That means that the ingredients of a slow query are not present with an INDEX UNIQUE SCAN.

### Concatenated Indexes
* A concatenated index is one index across multiple columns.
* Database creates an index on all primary key columns—a so-called concatenated index. 
* Note that the column order of a concatenated index has great impact on its usability so it must be chosen carefully.
* Whenever a query uses the complete primary key, the database can use an INDEX UNIQUE SCAN—no matter how many columns the index has.
* When using only one of the indexed columns, the index will not be used. Instead it performs a TABLE ACCESS FULL.
    * As a result the database reads the entire table and evaluates every row against the where clause.
* But first index column is always usable for searching.
* The most important consideration when defining a concatenated index is how to choose the column order so it can be used as often as possible.
* In general, a database can use a concatenated index when searching with the leading (leftmost) columns. 
    * An index with three columns can be used when 
        * searching for the first column
        * searching with the first two columns together
        * searching using all columns.
* Even though the two-index solution delivers very good select performance as well, the single-index solution is preferable. 
    * It not only saves storage space, but also the maintenance overhead for the second index. 
* The fewer indexes a table has, the better the insert, delete and update performance.
* To define an optimal index you must understand more than just how indexes work—you must also know how the application queries the data. 
    * This means you have to know the column combinations that appear in the where clause.      
 
## Slow Indexes, Part II
*  This section explains the way databases pick an index and demonstrates the possible side effects when changing existing indexes.

### The Query Optimizer
* The query optimizer, or query planner, is the database component that transforms an SQL statement into an execution plan. 
* This process is also called compiling or parsing. There are two distinct optimizer types.

* Cost-based optimizers (CBO)
    * Generate many execution plan variations and calculate a cost value for each plan. 
    * The cost calculation is based on the operations in use and the estimated row numbers. 
    * In the end the cost value serves as the benchmark for picking the “best” execution plan.

* Rule-based optimizers (RBO)
    * Generate the execution plan using a hard-coded rule set. 
    * Rule based optimizers are less flexible and are seldom used today.
    
## Execution Plans
* https://use-the-index-luke.com/sql/explain-plan

* How to run execution plan
    ```
    EXPLAIN PLAN FOR $SQL_STATEMENT;
    SELECT * FROM TABLE(dbms_xplan.display);
    ```


### Oracle SQL execution plan cost column tips[3]
* The cost column is supposed to be a guess of the number of single block disk reads required, but it's not very useful for SQL tuning.



## Functions 

### Case-Insensitive Search with Function-Based Indexes 

#### Using UPPER or LOWER with normal index
* Table employee has an index on LAST_NAME column.
* SELECT first_name, last_name, phone_number FROM employee WHERE UPPER(last_name) = UPPER('winand')
* Database will use TABLE ACCESS FULL operation with filter-predicate(UPPER("LAST_NAME")='WINAND').

#### Using UPPER or LOWER with function-based index
* The database can use a function-based index if the exact expression of the index definition appears in an SQL statement.
* CREATE INDEX emp_up_name ON employees (UPPER(last_name))
* SELECT first_name, last_name, phone_number FROM employee WHERE UPPER(last_name) = UPPER('winand')
* Database will use TABLE ACCESS BY INDEX ROWID and INDEX RANGE SCAN operations with access-predicate(UPPER("LAST_NAME")='WINAND').

#### Warning - Sometimes ORM tools use UPPER and LOWER without the developer’s knowledge.
* Hibernate, for example, [injects an implicit](https://use-the-index-luke.com/sql/myth-directory/dynamic-sql-is-slow#myth-dynamic-sql-sample) LOWER for case-insensitive searches.

#### [Oracle Statistics for Function-Based Indexes](https://use-the-index-luke.com/sql/where-clause/functions/case-insensitive-search#sb-collecting-statistics)

### User-Defined Functions
* Function-based indexing is a very generic approach. Besides functions like UPPER you can also index expressions like A + B and even use user-defined functions in the index definition.
* There is one important exception. It is, for example, not possible to refer to the current time in an index definition, neither directly nor indirectly.
	* e.g. The function caculate the age with sysdate, the random number generators.
	* The reason behind this limitation is simple. When inserting a new row, the database calls the function and stores the result in the index and there it stays, unchanged. The database updates the indexed value only when the date is changed by an update statement. 

### Over-Indexing
* Every index causes ongoing maintenance. Function-based indexes are particularly troublesome because they make it very easy to create redundant indexes.
* To make one index suffice, you should consistently use the same function throughout your application.
* See also [Chapter 8, “Modifying Data”](https://use-the-index-luke.com/sql/dml). 

## Parameterized Queries

* Bind parameters—also called dynamic parameters or bind variables—are an alternative way to pass data to the database. Instead of putting the values directly into the SQL statement, you just use a placeholder like ?, :name or @name and provide the actual values using a separate API call.

### There are two good reasons to use bind parameters in programs:
* Security: Bind variables are the best way to prevent SQL injection.
* Performance:
	* Databases with an execution plan cache like SQL Server and the Oracle database can reuse an execution plan when executing the same statement multiple times. 
	* It saves effort in rebuilding the execution plan but works only if the SQL statement is `exactly` the same. If you put different values into the SQL statement, the database handles it like a different statement and recreates the execution plan.
	* When using bind parameters you do not write the actual values but instead insert placeholders into the SQL statement. That way the statements do not change when executing them with different values. 


### References 
* [1][Slow Indexes, Part I](https://use-the-index-luke.com/sql/anatomy/slow-indexes)
* [2][The Equality Operator](https://use-the-index-luke.com/sql/where-clause/the-equals-operator)
* [3][Oracle SQL execution plan cost column tips](http://www.dba-oracle.com/t_sql_execution_plan_cost_column.htm)
* [4][Introduction to Database System by Shan-Hung Wu](https://www.youtube.com/playlist?list=PLS0SUwlYe8cyln89Srqmmlw42CiCBT6Zn)
    


