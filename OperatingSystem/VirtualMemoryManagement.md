# Virtual Memory Management
## Background
* Why don't we want to run a program that is entirely in memory? -> `We want better memory utilization`(Increase the access times)
	* Many code for handling unusual errors or conditions
	* Certain program routines or features are rarely used
	* The same library code used by many programs
	* Arrays, lists and tables allocated but not used

* Virtual memory - separation of user logical memory from physical memory
	* To run a extremely `large process`: Logical address space can be much larger than physical address space
	* To increase `CPU/resources utilization`: Higher degree of multiprogramming degree
	* To `simplify programming` tasks: Free programmer from memory limitation
	* To run programs `faster`: Less I/O would be needed to load or swap

* Virtual memory can be implemented via
	* `Demand paging`
	* Demand segmentation: more complicated due to variable sizes
	* Since the size of segmentation will not be fixed. Most of the time -> Demand paging. 
		* Paging: fixed size, O(1) to find the empty space
		* Segmentation: Non-fixed size, O(N) to find the empty space

## `Demand Paging`
* A page rather than the whole process is brought into memory only when it is needed
	* Less I/O needed -> Faster response
	* Less memory needed -> More users

* Page is needed when there is a reference to the page
	* Invalid reference -> abort
	* Not-in-memory -> bring to memory via paging

* `Pure demand paging`
	* Start a process with no page
	* Never bring a page into memory until it is required.

* A `swapper`(midterm scheduler) manipulates the entire `process`, whereas a `pager` is concerned with the individual pages of a process
* Hardware support
	* `Page Table`: `a valid-invalid bit`
		* 1 -> page in memory
		* 0 -> page not in memory
		* Initially, all such bits are set to 0
	* Secondary memory(swap space, `backing store`): Usually, a high-speed disk(swap device) is use

* Page Fault
	* First reference to a page will trap to OS -> `page-fault trap`
	1. OS looks at the internal table (in PCB) to decide
		* Invalid reference -> abort
		* Just not in memory -> continue
	2. Get an empty frame
	3. Swap the page from disk(swap space) into the frame

* Page Replacement
	* If there is no free frame when a page fault occurs
		* Swap a frame to backing store
		* Swap a page from backing store into the frame
		* Difference page `replacement algorithms` pick different frames for replacement

* Demand Paging Performance
	* `Effective Access Time`(EAT): `(1-p) * md + p * pft`
		* P: `page fault rate`, ma: mem, access time, pft: page fault time
	* Example: ma = 200ns, pft = 8ms
		* EAT = (1 - p) * 200ns + p * 8ms = 200ns + 7,999,800ns * p
	* `Access time is proportional to the page fault rate`
		* If one access out of 1000 causes a page fault, then EAT = 8.2 microseconds -> `slowdown by a factor of 40!`
		* For degradation less than 10%: 200 > 200 + 7,999,800 * p, `p < 0.0000025` -> one access out of 399,990 to page fault

* Demand Paging Performance(Con't)
	* Programs tend to have `locality` of reference
	* Locality means program often accesses memory addresses that are close together
		* `A single page fault can bring in 4KB memory content`
		* Greatly reduce the occurrence of page fault
	* Major components of page fault time(about 8 ms)
		1. serve the page-fault interrupt
		2. `read in page from disk(most expensive)`
		3. restart the process
			* The 1st and 3rd can be reduced to several hundred instructions
		 	* The page switch time is close to 8ms

# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/watch?v=TnmN_KNIbc4&list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX&index=36)