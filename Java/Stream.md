## Java Stream

### Abstraction of Functions
* Stream is not d data structure, it is an abstraction of functions.
    * Bucket(List/Set) v.s. Pipeline(Stream)

### Stream operations and pipelines [1]
* Intermediate operations: such as filter or map. 
    * They are always lazy; executing an intermediate operation does not actually perform any filtering, but instead creates a new stream that, when traversed, contains the elements of the initial stream that match the given predicate. 
    * Traversal of the pipeline source does not begin until the terminal operation of the pipeline is executed.
    * Stateless operations: such as filter and map.
        * Stateless operations retain no state from previously seen element when processing a new element -- each element can be processed independently of operations on other elements. 
        
    * Stateful operations: such as distinct and sorted, 
        * Stateful operations may incorporate state from previously seen elements when processing new elements.
        * Stateful operations may need to process the entire input before producing a result. 
            * For example, one cannot produce any results from sorting a stream until one has seen all elements of the stream. 
            * As a result, under parallel computation, some pipelines containing stateful intermediate operations may require multiple passes on the data or may need to buffer significant data. 
            * Pipelines containing exclusively stateless intermediate operations can be processed in a single pass, whether sequential or parallel, with minimal data buffering.
        
* Terminal operation: such as forEach or reduce.
    * Terminal operations may traverse the stream to produce a result or a side-effect. 
    * After the terminal operation is performed, the stream pipeline is considered consumed, and can no longer be used.

### Parallelism[1]
* Streams facilitate parallel execution by reframing the computation as a pipeline of aggregate operations, rather than as imperative operations on each individual element. 
* All streams operations can execute either in serial or in parallel.
* Collection.stream() and Collection.parallelStream(),  BaseStream.sequential() and BaseStream.parallel(), isParallel().
    * Note - last call wins[3]:
	```
    .parallel()
	.sequential()
	.parallel()
	.sequential()
    ```
* Except for operations identified as explicitly non-deterministic, such as findAny(), whether a stream executes sequentially or in parallel should not change the result of the computation.
* Most stream operations accept parameters that describe user-specified behavior, which are often lambda expressions. To preserve correct behavior, these behavioral parameters must be `non-interfering`, and in most cases must be `stateless`.
* Parallel and reduce[3]
	* Use reduce with parallel will execute extra times than sequential?
	* Should use identity value(e.g 0) instead of the non-identity value(e.g. 21)
	```
    .reduce(0, (total, e) -> add(total, e)) + 21;
    ```   
        
### Non-interference[1]
* Streams enable you to execute possibly-parallel aggregate operations over a variety of data sources, including even non-thread-safe collections such as ArrayList.
* This is possible only if we can prevent interference with the data source during the execution of a stream pipeline.
* For most data sources, preventing interference means ensuring that the data source is not modified at all during the execution of the stream pipeline.
* The notable exception to this are streams whose sources are concurrent collections, which are specifically designed to handle concurrent modification. Concurrent stream sources are those whose `Spliterator` reports the `CONCURRENT` characteristic.

### Stateless behaviors[1]
Stream pipeline results may be nondeterministic or incorrect if the behavioral parameters to the stream operations are stateful. 
	```
     Set<Integer> seen = Collections.synchronizedSet(new HashSet<>());
     stream.parallel().map(e -> { if (seen.add(e)) return 0; else return e; })...
     ```
 	* If the mapping operation is performed in parallel, the results for the same input could vary from run to run, due to thread scheduling differences.
* Note also that attempting to access mutable state from behavioral parameters presents you with a bad choice with respect to safety and performance; 
	* Do not synchronize access to that state -> data race.
	* Do synchronize access to that state ->  you risk having contention undermine the parallelism you are seeking to benefit from. 

* The best approach is to use stateless behavioral parameters, stateless lambda expression.
	 
### Laziness[3]
* Stream does not execute a function on a collection of data
* Stream instead executes a collection of functions on a piece of data
* FindFirst -> will not go through all data

### Sequential Code v.s. Concurrent Code[3]
* With Stream, the structure of sequential code is the same as the structure of concurrent code.(e.g. parallelStream(), parallel())

### Limitations of Java Steam[2]
* Single use only(IllegalStateException: stream has already been operated upon or closed.)
    * We can not reuse a stream. You can only use it once and should not share it.
* Single pipeline - a single terminal operation
* Hard to Handle Exception 
    * Scala and Haskell has interfaces to handle exceptions

## References
* [1][Package java.util.stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)
* [2][Java Streams vs Reactive Streams: Which, When, How, and Why?](https://github.com/EddieChoCho/tech-talks-note/blob/master/2018/JavaStreamsVsReactiveStreamsWhichWhenHowAndWhy.md)
* [3][Parallel and Asynchronous Programming with Streams and CompletableFuture](https://github.com/EddieChoCho/tech-talks-note/blob/master/2017/ParallelAndAsynchronousProgrammingWithStreamsAndCompletableFuture.md)