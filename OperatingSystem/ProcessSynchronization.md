# Process Synchronization

## Background
* `Concurrent` access to shared data may result in `data inconsistency`
* Maintaining data consistency requires mechanism to `ensure the orderly execution` of cooperating processes

### Example: Concurrent Operations on counter 
* The statement "counter++" may be implemented in machine language as:
	```
	move ax, counter
	add  ax, 1
	move counter, ax
	```
* The statement "counter--" may be implemented  as:
	```
	move bx, counter
	add  bx, 1
	move counter, bx
	```

#### Instruction Interleaving
* Assume the counter is initially 5. One interleaving of statement os:
	```
	producer: move ax, counter    -> ax = 5
	producer: add ax, 1           -> ax = 6
	context switch
	consumer: move bx, counter    -> bx = 5
	consumer: sub bx, 1           -> bx = 4
	context switch
	producer: move counter, ax    -> counter = 6
	context switch
	producer: move counter, bx    -> counter = 4 (wrong)
	```
* The value of the counter may be either 4, 5, or 6, where the correct result should be 5.
* Happens with Preemptive scheduling.

### Race Condition
* `Race condition`: 
	* The situation where several process `access and manipulate shared data concurrently`
	* The final value of the shared data `depends upon which process finished last`
* To prevent race condition, concurrent processes must be `synchronized`
	* On a single-processor machine, we could `disable interrupt` or use `non-preemptive CPU scheduling`
* Commonly described as `critical section problem`

# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/watch?v=CNO_I8jhX3I&list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX&index=52)