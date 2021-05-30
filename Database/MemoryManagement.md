# Memory Management
* Consequences of Slow I/Os
  * Architecture that minimizes I/Os:
     * Block access to/from disks
     * Self-managed caching of blocks
     * Choose the plan that costs least(fewest block I/Os)

* Storage Access Patterns
  * `Spatial locality`: each client(e.g. scan) focuses on a small number of blocks a time
     * Despite ending up with huge block accesses. e.g. To produce the next output record, a product scan needs only two blocks a time(left and right)

  * `Temporal locality`: recently used blocks are likely to be used in the near future. e.g. blocks of catalogs

## Minimizing Disk Access by Caching
* Idea: to reserve a pool of pages that keep the contents of most currently used blocks
  * To swap in/out blocks only when there's no empty page left in the pool

* Benefits
  * Economic: only small memory space required
  * Saves reads(if a requested block hits a page)
  * Save writes: all values set to a block only need to be written once upon swapping

* Why not use virtual memory(provided by OS)?
  * Virtual Memory: a very large address space for each process(larger than physical memory)
  * Problem 1: bad page replacement algorithms. e.g. FIFO, LRU, etc
     * OS has no idea which blocks will definitely be used by a process in the near future
     * E.g. In a product scan, DBMS knows it's best to hold a left-child block at all times during scanning the right-child(as a select). But OS doesn't

  * Problem 2: uncontrolled delayed writes because of automatic swapping
     * When powered off, dirty pages may gone
        * Impairs the DBMS ability to recover after a system crash
        * Hurts durability, the D of the ACID, of committed transactions
     * Immediate writes?
        * Impairs the caching
        * Data may still corrupt due to partial writes upon crash
* Meta-writes(of logs) are needed

* Self-Managed Page in DBMS
  * Pro 1: Controlled swapping
     * Fewer I/Os than VM via better replacement strategy
     * DBMS can tell which page must/cannot be flushed
  * Pro 2: Supports meta-writes
     * DBMS can write logs to recover from crashes

* What Blocks to Cache?
  * Those of user data(DB, including catalogs)
     * Pages for these blocks are managed by the `buffer manager`
  * Those of logs
     * In meta-writes
     * Pages managed by the `log manager`

## Buffering User Data
* Access Pattern to User Blocks
  * `Random` block reads and writes
     * From clients directly
     * Even from sequential scans(if above OS file system. Then for OS, it is random access)
  * Concurrent access to multiple blocks by multiple`threads(Threading issue, each thread per, e.g. JDBC client)
  * `Predictable` access to certain blocks
     * Each scan needs certain blocks a time
     * In particular, a table scan need one block a time and can forget what just read

* To reduce I/Os, the buffer manager allocates a pool of pages, called `buffer pool`
  * Caching multiple blocks
  * Implement swapping
* Pages should be the direct I/O buffers held by the OS
  * Advoids swapping by VM
  * Eliminates the redundancy of double buffering
  * e.g. ByteBuffer.allocateDirect()

### How to make use of predictable block accesses to further reduce I/Os?
* Each table scan needs one block a time
  * The semantic of blocks is hidden behind the associated RecordFile
* It is the RecordFile instances that talk to the memory manager about which blocks are needed. One instance per thread/client
* Through pinning
  * When a RecordFile needs a block
     1. Asks buffer manager to `pin`(read-in) a block in some page
     2. Client accesses the contents of the page
     3. When the client is done with the block, it tells the buffer manager to `unpin` the block
  * When swapping, only pages containing the `unpinned` blocks can be swapped out

* Result of pinning
  * A hit, no I/O
  * Swapping: there exists at least one `candidate page` in the buffer pool holding unpinned block
     * Need to flush the page contents back to disk if the page is dirty
     * Which candidate page? `replacement strategies`
     * Then read in the desired block
  * Waiting: If all pages in the buffer pool are pinned. Wait until some other unpins a page

# References
* [Introduction to Database System by Shan-Hung Wu](https://www.youtube.com/playlist?list=PLS0SUwlYe8cyln89Srqmmlw42CiCBT6Zn)
