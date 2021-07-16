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

# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/watch?v=efH4nuwUalA&list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX&index=62)