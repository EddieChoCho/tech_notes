# Deadlock
* Deadlock Problem
	* A set of blocked processes each `holding` some resources and `waiting` to acquire a resource held by another process in the set.

## Necessary Conditions
* `Mutual exclusion`
	* only 1 process at a time can use a resource
* `Hold & Wait`
	* a process holding some resource and is waiting for another resource
* `No preemption`
	* a resource can be only released by a process voluntarily
* `Circular wait`
	* there exists a set {P0, P1,...,Pn} of waiting process such that P0 -> P1 -> ...-> Pn -> P0

## System Model
* Resources types R1, R2, ..., Rm
	* e.g. CPU, memory pages, I/O devices
* Each resource type Ri has `Wi instances`
	* e.g. a computer has 2 CPUs
* Each process utilizes a resource as follows:
	* `Request` -> `use` -> `release`

## Resource-Allocation Graph

## Deadlock Detection
* If a graph contains `no cycle -> no deadlock`
	* `Circular wait` cannot be held
* If a graph contains a cycle:
	* If `one instance per resource type` -> deadlock
	* if `multiple instances per resource type` -> `possibility` of deadlock

## Handling Deadlock
* Ensure the system will `never enter a deadlock state`
	* `deadlock prevention:` ensure that at least one of the `4 necessary conditions` cannot hold
	* `deadlock avoidance:` `dynamically` examines the resource-allocation state before allocation

* Allow to `enter a deadlock state` and `then recover`
	* `deadlock detection`
	* `deadlock recovery`

* Ignore the problem and pretend that deadlocks never occur in the system
	* used by most operating systems, including UNIX.

### Deadlock Prevention
* `Mutual exclusion(ME)`: do not require ME on sharable resources
	* e.g. there is no need to ensure ME on read-only files
	* `Some resources are not shareable, however`(e.g. printer)

* `Hold & Wait`:
	* When a process requests a resource, it does not hold any resource
	* Pre-allocate all resources before executing
	* `Cons: resource utilization is low; starvation is possible`

### Deadlock Prevention(con't)
* `No preemption`
	* When a process is waiting on a resource, all its holding resources are preempted
		* e.g. P1 requests R1 which is allocated to P2, which in turn is waiting on R2. (P1 -> R1 -> P2 -> R2)
		* `R1 can be preempted and reallocated to P1`
	* Applied to resource whose states can be easily saved and restored later
		* e.g. CPU register & memory
	* `It cannot easily be applied to other resources.`
		* `e.g. printer & tape derives` 

* `Circular wait`
	* impose a total ordering of all resources types
	* a process requests resources in an increasing order 
		* Let R = {R0, R1, ..., Rn} be the set of resource types
		* `When request Rk, should release all Ri, i >= k`
	* Example:
		* F(tape drive) = 1, F(disk drive) = 5, F(printer) = 12
		* A process must request tape and disk drive before printer

## Deadlock Avoidance
* `safe state`: a system is in a safe state if there exists `a sequence of allocations` to satisfy requests by all process
	* THe sequence of allocations is called `safe sequence`
* safe state -> no deadlock
* unsafe state -> `possibility` of deadlock
* deadlock avoidance -> `ensure that a system never enters an` unsafe state
* check the worst case

#### Safe state with safe sequence
#### Un-Safe state w/o safe sequence

### Avoidance Algorithms
* `Single instance` of a resource type
	* `resource-allocation graph(RAG) algorithm` based on circle detection

#### Resource-Allocation Graph(RAG) Algorithm
* `Request edge`
* `Assignment edge`
* `Claim edge`
	* `Claim edge` converts to `request edge`: When a `resource is requested` by a process
	* `Assignment edge` converts back to a `claim edge`: When a `resource is released` by a process

* Resources `must be claimed a priori` in the system
* `Grant a request` only if `No cycle created`
* Check for safety using a `cycle-detection algorithm`


#### Banker's Algorithm
* Use for `multiple instances` of each resource type
* Based on safe sequence detection
* Use a general safety algorithm to `pre-determine` if any `safe sequence` exists after allocation
* Only proceed the allocation if safe sequence exists
* Safety algorithm
	1. Assume processes need `maximum` resources
	2. Find a process that can be satisfied by free resources
	3. Free the resource usage of the process
	4. Repeat to step 2 until all processes are satisfied

## Deadlock Detection
* `Single instance` of each resource type
	* convert request/assignment edges into wait-for graph
	* deadlock exists if there is a cycle in the wait-for graph

* `Multiple-Instance` for each resource type
	* Check if system is in a safe state -> no dead lock

## Deadlock Recovery
* `Process termination`
	* abort all deadlocked processes
	* abort 1 process at a time until the deadlock cycle is eliminated
		* which process should we abort first?

* `Resource preemption`
	* select a victim: which one to preempt?
	* rollback: partial rollback or total rollback?
	* starvation: can the same process be preempted always?

# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/watch?v=efH4nuwUalA&list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX&index=62)