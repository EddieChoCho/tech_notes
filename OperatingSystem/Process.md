# Process Concept 

* Program - passive entity: binary stored in disk
* Process - active entity: s program in execution in memory
* A process includes
	* Code segment(text section)
	* Data section - global variables
	* Stack - temporary local variables and functions
	* Heap - dynamic allocated variables or classes
	* Current activity(program counter, register contents)
		* A program counter is a register in a computer processor that contains the address (location) of the instruction being executed at the current time.
	* A set of associated resources(e.g. open file handlers)

* Threads
	* A.k.a lightweight process: basic unit of CPU utilization
	* All threads belonging to the same process share: code section, data section, and OS resources(e.g. open files)
	* But each thread has it own: thread ID, program counter, register set, and stack

* Process/Thread States
	* New: the process is being created
	* Ready: the process is in the memory waiting to be assigned to a processor
	* Running: instructions are being executed by CPU
	* Waiting: the process is waiting for events to occur
	* Terminated: the process has finished execution

	* Only one process is `running` on any processor at any instant
	* However, many process may be `ready` or `waiting`

* Process Control Block(PCB): Info. associated with each process
	* Process state
	* Program counter
	* CPU registers
	* CPU scheduling information(e.g. priority)
	* Memory-management information(e.g. base/limit register)
	* I/O status information
	* Accounting information
	* ...

* Context Switch
	* Kernel save the state of the old process(PCB), and loads the saved state(PCB) for the new process.
	* Context-Switch time is purely overhead
	* Switch time(about 1~1000 ms) depends on
		* memory speed
		* number of registers
		* existence of special instructions
			* a single instruction to save/load all registers
		* hardware support
			* multiple set of registers(Sun UltraSPARC - a context switch means changing register file pointer).


## Process Scheduling
* Multiprogramming: CPU runs process at all times to `maximize CPU utilization`
* Time sharing: switch CPU frequently such that `user` can `interact` with each program while it is running
* Processes will have to wait until the CPU is free and can be rescheduled

* Process Scheduling Queues
	* Processes migrate between the various queue(e.g. switch among states)
	* Job queue(New State) - set of `all processes` in the system
	* Ready queue(Ready State) - set of all processes residing in main memory, `ready and waiting to execute`
	* Device queue(Wait State) - set of processes `waiting for an I/O device`

### Schedulers
* Short-term scheduler(`CPU scheduler`) - selects which process should be executed and `allocated CUP(Ready state -> Run state)`
* Long-term scheduler(`job scheduler`) - select which processes should be `loaded into memory` and brought into the ready queue`(New state -> Ready state)`
* Medium-term scheduler - select which processes should be `swapped in/out memory(Ready state -> Wait state)`

#### Long-Term scheduler
* Control degree of multiprogramming
* Execute less frequently(e.g. invoked only when a process leaves the system or `once several minutes`)
* Select a `good mix of of CPU-bound & I/O-bound` processes to increase system overall performance
* UNIX/NT: no long-term scheduler
	* Created process placed in memory for short-term scheduler
	* Multiprogramming degree is bounded by hardware limitation or on the self-adjusting nature of users

#### Short-Term Scheduler
* Execute quite frequently(e.g. once per 100ms)
* The scheduling algorithm must be efficient to reduce the overhead

#### Medium-Term Scheduler
* swap in/out between memory and disk
* `swap out`: removing processes from memory to reduce the degree of multiprogramming
* `swap in`: reintroducing swap-out processes into memory
* Purpose: improve process mix, free up memory
* Most modern OS doesn't have medium-term scheduler because having sufficient physical memory or using virtual memory

## Operations on Processes

* Tree of Processes
	* pid: each process is identified by a unique processor identifier

### Process Creation
* Resource sharing
	* Parent and child processes share all resources
	* Child process shares subset of parent's resources
	* Parent and child share no resources

* Two possibilities of execution
	* Parent and children `execute concurrently`
	* Parent `waits until children terminate`

* Two possibilities of address space
	* `Child duplicate of parent`, communication via sharing variables
	* `Child has a program loaded into it`, communication vua message passing

#### UNIX/Linux Process Creation
* `fork` system call
	* Create a new(child) process
	* The new process `duplicates the address space` of it's parent
	* Child & Parent `execute concurrently` after fork
	* Child: return value of fork is 0
	* Parent: return value of fork is PID of the child process

* `execlp` system call
	* `Load a new binary file` into memory - `destroy the old code`
* `wait` system call
	* The parent waits for `one of its child processes` to complete

* Memory space of fork():
	Old implementation: Child is an `exact copy` of parent
	Current implementation: use `copy-on-wrote` technique to store `differences in child address space`

### Process Termination
* Terminate when the last statement is executed or `exit()` is called
	* All resources of the process, including physical & virtual memory, open files, I/O buffers are `deallocated by the OS`

* Parent may terminate execution of children processes by specifying its PID (`abort`)
* Cascading termination: killing(exiting) parent -> killing(exiting) all its children


# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/playlist?list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX)