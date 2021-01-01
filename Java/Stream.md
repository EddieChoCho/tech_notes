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