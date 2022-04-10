# Explain Plan - Postgresql

## 1. EXPLAIN Basics

* The structure of a query plan is a tree of plan nodes.
    * Nodes at the bottom level of the tree are scan nodes: they return raw rows from a table.
    * There are different types of scan nodes for different table access methods: sequential scans, index scans, and
      bitmap index scans.

* There are also non-table row sources, such as VALUES clauses and set-returning functions in FROM, which have their own
  scan node types.
* If the query requires joining, aggregation, sorting, or other operations on the raw rows, then there will be
  additional nodes above the scan nodes to perform these operations.

* e.g. ,
    ```sql
        EXPLAIN SELECT * FROM tenk1;

                         QUERY PLAN
        -------------------------------------------------------------
        Seq Scan on tenk1  (cost=0.00..458.00 rows=10000 width=244)
    ```
    * Since this query has no WHERE clause, it must scan all the rows of the table, so the planner has chosen to use a
      simple sequential scan plan(Seq Scan on tenk1).
    * Estimated start-up cost(`cost=0.00`..458.00). This is the time expended before the output phase can begin, e.g.,
      time to do the sorting in a sort node.
    * Estimated total cost(cost=0.00..`458.00`). This is stated on the assumption that the plan node is run to
      completion, i.e., all available rows are retrieved. In practice a node's parent node might stop short of reading
      all available rows.
        * e.g., tenk1 has 358 disk pages and 10000 rows. The estimated cost is computed as (disk pages read
          * `seq_page_cost`) + (rows scanned * `cpu_tuple_cost`).
        * By default, seq_page_cost is 1.0 and cpu_tuple_cost is 0.01, so the estimated cost is (358 * 1.0) + (10000 *
          0.01) = 458.

    * Estimated number of rows output by this plan node(rows=10000). Again, the node is assumed to be run to completion.
    * Estimated average width of rows output by this plan node (width=244, in bytes).

* cost
    * cost parameters
        * The costs are measured in arbitrary units determined by the planner's cost parameters.
        * Traditional practice is to measure the costs in units of disk page fetches; that is, seq_page_cost is
          conventionally set to 1.0 and the other cost parameters are set relative to that.

    * cost of an upper-level node includes the cost of all its child nodes.
    * cost only reflects things that the planner cares about.
        * In particular, the cost does not consider the time spent transmitting result rows to the client, which could
          be an important factor in the real elapsed time; but the planner ignores it because it cannot change it by
          altering the plan.

* rows
    * rows value is the number of rows emitted by the node(not the number of rows processed or scanned by the plan node)
      .
    * This is often less than the number scanned, as a result of filtering by any WHERE-clause conditions that are being
      applied at the node.
    * Ideally the top-level rows estimate will approximate the number of rows actually returned, updated, or deleted by
      the query.

* TBD...

## 2. Operations

### 2-1. Index and Table Access

#### Seq Scan

* The Seq Scan operation scans the entire relation (table) as stored on disk (like TABLE ACCESS FULL).

#### Index Scan

* The Index Scan performs a B-tree traversal, walks through the leaf nodes to find all matching entries, and fetches the
  corresponding table data.
* It is like an INDEX RANGE SCAN followed by a TABLE ACCESS BY INDEX ROWID operation.
* Note: The so-called index filter predicates often cause performance problems for an Index Scan.(TBD...)

#### Index Only Scan

* The Index Only Scan performs a B-tree traversal and walks through the leaf nodes to find all matching entries.
* There is no table access needed because the index has all columns to satisfy the query (exception: MVCC visibility
  information).

#### Bitmap Index Scan / Bitmap Heap Scan / Recheck Cond

* A plain Index Scan fetches one tuple-pointer at a time from the index, and immediately visits that tuple in the table.
* A bitmap scan fetches all the tuple-pointers from the index in one go, sorts them using an in-memory “bitmap” data
  structure, and then visits the table tuples in physical tuple-location order.

### 2-2. Join Operations

* Generally join operations process only two tables at a time. In case a query has more joins, they are executed
  sequentially: first two tables, then the intermediate result with the next table.
* In the context of joins, the term “table” could therefore also mean “intermediate result”.

#### Nested Loops

* Joins two tables by fetching the result from one table and querying the other table for each row from the first.

#### Hash Join / Hash

* The hash join loads the candidate records from one side of the join into a hash table (marked with Hash in the plan)
  which is then probed for each record from the other side of the join.

#### Merge Join

* The (sort) merge join combines two sorted lists like a zipper.
* Both sides of the join must be presorted.

### 2-3. Sorting and Grouping

#### Sort / Sort Key

* Sorts the set on the columns mentioned in Sort Key.
* The Sort operation needs large amounts of memory to materialize the intermediate result (not pipelined).
* Further reading: Indexing Order By

#### GroupAggregate

* Aggregates a presorted set according to the group by clause.
* This operation does not buffer large amounts of data (pipelined)
* Further reading: Indexing Group By

#### HashAggregate

* Uses a temporary hash table to group records.
* The HashAggregate operation does not require a presorted data set, instead it uses large amounts of memory to
  materialize the intermediate result (not pipelined).
* The output is not ordered in any meaningful way.
* Further reading: Indexing Group By

### 2-4. Top-N Queries

#### Limit

* Aborts the underlying operations when the desired number of rows has been fetched.
* The efficiency of the top-N query depends on the execution mode of the underlying operations.
* It is very inefficient when aborting non-pipelined operations such as Sort.

#### WindowAgg

* Indicates the use of window functions.

## 3. Distinguishing Access and Filter-Predicates

* The PostgreSQL database uses three different methods to apply where clauses (predicates)
* PostgreSQL execution plans do not show index access and filter predicates separately—both show up as “Index Cond”.
    * That means the execution plan must be compared to the index definition to differentiate access predicates from
      index filter predicates.

#### Access Predicate (“Index Cond”)

* The access predicates express the start and stop conditions of the leaf node traversal.

#### Index Filter Predicate (“Index Cond”)

* Index filter predicates are applied during the leaf node traversal only.
* They do not contribute to the start and stop conditions and do not narrow the scanned range.

#### Table level filter predicate (“Filter”)

* Predicates on columns that are not part of the index are evaluated on the table level.
* For that to happen, the database must load the row from the heap table first.

## References

* [Postgresql - Chapter 14. Performance Tips](https://www.postgresql.org/docs/14/performance-tips.html)
* [Use the index, Luke! Explain-plan - Postgresql](https://use-the-index-luke.com/sql/explain-plan/postgresql)
* [Reading a Postgres EXPLAIN ANALYZE Query Plan](https://thoughtbot.com/blog/reading-an-explain-analyze-query-plan)