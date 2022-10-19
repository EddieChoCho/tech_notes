# Query Processing and Optimization

## Pipelining and Materialization

* Pipelining(or on-the-fly processing)
    * The result of an operator is sometimes pipelined to another operator without creating a temporary relation to hold
      the intermediate result.
    * Pipelining is sometimes used to improve the performance of the queries, which can eliminate the cost of reading
      and writing temporary relations.

* Materialization
    * The materialization process starts from the lowest level operations in the expression.
    * The output of a materialized operation is stored in a temporary relation (stored on the secondary storage or disk)
      for processing for the next operation.

* The efficiency of the query evaluation can be improved by reducing the number of temporary files that are produced.
* Therefore, several relational operations are combined into a pipeline of operations in which the results of one
  operation is pipelined to another operation without creating a temporary relation to hold the intermediate result.

* Advantages
    * The use of pipelining saves on the cost of creating temporary relations and reading the result back in again.
* Disadvantages
    * The inputs to operations are not necessarily available all at once for processing.
    * This can restrict the choice of algorithms.

# Query Optimization

* A good scan tree can be faster than a bad one for orders of magnitude
* Query optimizer
    1. Generate candidate plan trees
    2. Estimate cost of each corresponding scan tree(not discussed yet)
    3. Pick and open the "best" one to execute query
* Goal:
    * Ideally: find the one with least cost
    * In practice: avoid bad trees

## Metric for Cost

* Cost pf a query
    * To user: query delay
    * Low delay also implies better system throughput
    * Typically, I/O delay dominates query delay

* Cost Estimation
    * For each plan/table p, we estimate B(p)
        * number of blocks accessed by the corresponding scan

    * Usually estimating B(q) requires ,ore knowledge:
        * R(p): number of records output
        * Search cost: number of blocks of index, if used
        * V(p,f): number of distinct values for field f in p

    * Estimating B(P)
        * TBD...(add table content)

* For Any p, We need to estimate R(p) and Index Search Cost
    * Index Search Cost:
        * HashIndex.searchCost()
        * BTreeINdex.searchCost()

    * Estimating R(p) is called cardinality estimation

## References

* Database Systems: Concepts, Design and Applications by S. K. Singh
* [14.528 Query Execution Models, Function Calls vs Pipelining, Pipeline Breakers by Prof. Dr. Jens Dittrich](https://youtu.be/SR4AaqA4DXY)
