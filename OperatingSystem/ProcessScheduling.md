# Process Scheduling
* CPU scheduler

## Basic Concepts
* The idea of multiprogramming
	* Keep several processes in memory. Every time one process has to wait, another process takes over the use of the CPU
* `CPU-I/O burst cycle`: Process execution consists of a cycle of CPU execution and I/O wait(i.e., `CPU burst` and `I/O burst`).
	* `Generally, there is a large number of short CPU bursts, and a small number of long CPU bursts`
	* A I/O-bound program would typically has many very short CPU bursts
	* A CPU-bound program might have a few long CPU bursts

* CPU Scheduler 
	* Select from ready queue to execute(i.e. allocates a CPU for the selected process)

### Preemptive v.s. Non-preemptive
* CPU scheduling decisions may take place when a process:
	1. `Switches from running to waiting state` (e.g. CPU burst -> I/O burst)
	2. Switches from running to ready state (e.g. timer sharing/rescheduling)
	3. Switches from waiting to ready (e.g. I/O task finished -> new candidate in ready queue)
	4. `Terminates`

* `Non-preemptive` scheduling:
	* Scheduling under 1 and 4 (`no choice in terms of scheduling`)
	* The process keeps the CPU until it is `terminated` or `switched to the waiting` state
	* e.g Window 3.x

* `Preemptive` scheduling:
	* Scheduling under all cases
	* e.g. Windows 95 and subsequent versions, Mac OS X

* Preemptive Issues
	* Inconsistent state of shared data
		* Require `process synchronization`
		* Incurs a cost associated with access to shared data
	* Affect the design of OS kernel
		* the process is preempted in the middle of critical changes(for instance, I/O queues) and the kernel(or the device driver) needs to read or modify the same structure?
		* `Unix solution: waiting` either for a system call to complete or for an I/O block to take place before doing a context switch(`disable interrupt`)

### Dispatcher
* `Dispatcher` module gives control of the CPU to the process selected by scheduler
	* switching context
	* jumping to the proper location in the selected program
	
* `Dispatch latency`-time it takes for the dispatcher to stop one process and start another running
	* Scheduling time
	* Interrupt re-enabling time
	* Context switch time


# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/watch?v=EpZgvKBf5eo&list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX&index=47)