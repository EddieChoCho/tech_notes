# File System Interface
## File Concept
### File operations
* Create, write, read, reposition within a file(i.e. file seek), delete, truncate
* The metadata for managing the operations: Process-`open-file table`, OS-`system-wide table`
* Open-File Tables
	* Per-process table
		* Tracking all files opened by this process
		* Current `file pointer` for each opened file
		* `Access rights` and `accounting` information
	* System-wide table
		* Each entry in the per-process table points to this table
		* `Process-independent information` such as `disk location, access dates, file size`
		* `Open count`

* Open-File Attributes
	* Open-file attributes(metadata)
		* File pointer(per-process)
		* File open count(system table)
		* Disk location(system table)
		* Access rights(per-process)
	* File types 
		* .exe, .txt, etc
		* `Hint` for OS to operate file in a `reasonable` way

## Access Methods
* Sequential access
* Direct(relative) access
* Index access 

# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/watch?v=xzjOe7-m6qc&list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX)