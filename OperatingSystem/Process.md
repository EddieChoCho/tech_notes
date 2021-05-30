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

# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/playlist?list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX)