JVM Startup
Just-in-time(JIT) Compilation (dynamic compilation)

Solution 1: Ahead of Time(AOT) Compilation
AOT -> skip the bytecode, directly generate native code

*

AOT v.s. JIT

Solution 2: Store JIT Compilation Data

Solution 3: Decouple The JIT Compiler

Solution 4: Solution 2 + Solution 3

Co-ordinated Resume In Userspace(CRIU)

* Linux project

CRIU Challenges

* What if we move the saved state to a different machine?
  * What about open filed, shared memory, etc?
* What id we use the same state to restart multiple instances
  * They would work as if each was the original instance
  * Could lead to difficult clashes of use of state

* A JVM(and therefore Java code) would assume it was continuing its task
  * Very difficult to use effectively

Co-ordinated Restore at Checkpoint(CRaC)

* CRaC also enforces more restrictions on a checkpoint application
  * No open files or sockets
  * Checkpoint will be aborted if any are found


* [Java on CRaC: Superfast JVM Application Startup by Simon Ritter](https://youtu.be/bWmuqh6wHgE?si=SqLB_BXXa6uGAlbP)
* [Keeping Your Java Hot by Solving the JVM Warmup Problem By Simon Ritter](https://youtu.be/g6Y6F-UfYTI?si=QHSFuvfYvFxW1jvx)
