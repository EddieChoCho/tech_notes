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

## Scheduling Algorithms
* Scheduling Criteria
	* `CUP utilization`
		* theoretically: 0%~100%
		* real system: 40%(light)~90%(heavy)
	* `Throughput`
		* number of completed `processes` per time unit
	* `Turnarund time`
		* submission ~ completion
	* `Waiting time`
		* total waiting time int the `ready queue`
	* `Response time`
		* submission ~ the `first response` is produced


### Algorithms

#### First-Come, First-Served(FCFS) scheduling
* Convoy effect: short processes behind a long process

#### Shortest-Job-First(SJF) scheduling
* Associate with each process the length of its next CPU burst
* `A process with shortest burst length gets the CPU first`
* `SJF provides the minimum average waiting time(optimal!)`
* Two schemes
	* Non-preemptive - once CPU given to a process, it cannot be preempted until its completion
	* Preemptive - if a new process arrives with shorter burst length, preemption happens
	* Wait time = completion time - arrival time - run time(burst time)

* Approximate Shortest-Job-First(SJF)
	* SJF difficulty: `no way to know length of the next CPU burst`
	* `Approximate SJF`: the next burst can be `predicated` as an `exponential average` of the measured length of previous CPU bursts

#### Priority scheduling
* A priority number is associated with each process
* `The CPU is allocated to the highest priority process`
	* Preemptive
	* Non-preemptive
* SJF is a priority scheduling where priority is the predicted next CPU burst time
* Problem: starvation(low priority processes never execute)
	* e.g. IBM 7094 shutdown at 1973, and 1967-process never run
* `Solution: aging`(as time progresses increase the priority of processes)
	* e.g. increase priority by 1 every 15 minutes

#### Round-Robin scheduling
* Each process gets a small unit of CPU time(time quantum), usually 10 ~ 100 ms
* After TQ(Time Quantum) elapsed, process is `preempted` and added `to the end of the ready queue`
* Performance
	* TQ large -> `FIFO`
	* TQ small -> (context switch) `overhead` increases
* Typically, higher average turnaround than SJF, but better response

#### Multilevel queue scheduling
* Ready queue is partitioned into `separate queues`
* Each queue has its own scheduling algorithm
* `Scheduling must be done between queues`
	* Fixed priority scheduling: possibility of starvation
	* Time slice - each queue gets a certain amount of CPU(e.g. 80%, 20%)

#### Multilevel feedback queue scheduling
* A process can `move between the various queue`; `aging` can be implemented
* Idea: separate processes according to the characteristic of their CPU burst
	* I/O-bound and interactive processes in higher priority queue -> short CPU burst
	* CPU-bound processes in lower priority queue -> long CPU burst

* In general, multilevel feedback queue scheduler is defined by the following parameters:
	* Number of queues
	* Scheduling algorithm for each queue
	* Method used to determine when to upgrade a process
	* Method used to determine when to demote a process

### Evaluation Methods
* Deterministic modeling - takes a particular predetermined workload and defines the performance of each algorithm for that workload
	* Cannot be generalized

* Queueing model - mathematical analysis
* Simulation
* Implementation

# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/watch?v=EpZgvKBf5eo&list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX&index=47)