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

## Directory Structure
* Single-level directory
* Two-level directory
* Tree-structured directory
* Acyclic-graph directory
	* Use links to share files or directories
		* UNIX-like: `symbolic link`(ln -s/spell/count/dict/count)
	* A file can have `multiple absolute paths`
	* When does a file actually get deleted?
		* deleting the link but not the file
		* deleting the file but leaves the link -> `dangling pointer`
			* deleting the file when `reference counters` is 0

* General-graph directory
	* `May contain cycles`
	* How can we deal with cycles?
		* `Garbage collection`
			* First pass traverses the entire graph and marks accessible files or directories
			* Second pass collect and free everything that is unmarked

## File System Mounting
* A file system must be `mounted before` it can be `accessed`
* `Mount point`: `the root path` that a FS will be mounted to
* `Mount timing`:
	* `boot time`
	* `automatically at run-time`
	* `manually at run-time`
* Linux command: `mount -t` type device directory

## FIle Sharing
* File sharing on multiple users
	* Each user: (`userID`, `groupID`)
		* ID is associated with every ops/process/thread the user issues
	* Each file has 3 set of attributes
		* `owner, group, others`
	* Owner attributes describe the `privileges` for the owner of the file
		* same for group/others attributes
		* group/others attributes are set by `owner` or `root`

### Access-Control List
* We can create an `access-control list`(ACL) for `each user`
	* check requested file access against ACL
	* problem: unlimited # of users
* 3 classes of users -> 3 ACL(`RWX`) for `each file`
	* owner(e.g., 7 = RWX = 111)
	* group(e.g., 6 = RWX = 110)
	* public(others)(e.g., 4 = RWX = 100)

### File Protection
* File owner/creator should be able to control
	* what can be done
	* by whom
	-> Access control list
* Files should be kept from
	* physical damage(reliability): i.e. `RAID`
	* improper access(protection): i.e. password

# References
* [Operating System Course by Jerry Chou](https://www.youtube.com/watch?v=xzjOe7-m6qc&list=PLS0SUwlYe8czigQPzgJTH2rJtwm0LXvDX)