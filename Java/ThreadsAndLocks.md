## Chapter 17. Threads and Locks

### 17.1. Synchronization

### 17.2. Wait Sets and Notification

#### 17.2.1. Wait

#### 17.2.2. Notification

#### 17.2.3. Interruptions

#### 17.2.4. Interactions of Waits, Notification, and Interruption

### 17.3. Sleep and Yield

### 17.4. Memory Model

* Intra-thread semantics
    * The actions of each thread in isolation must behave as governed by the semantics of that thread, with the
      exception that the values seen by each read are determined by the memory model.
    * When we refer to this, we say that the program obeys intra-thread semantics. Intra-thread semantics are the
      semantics for single-threaded programs, and allow the complete prediction of the behavior of a thread based on the
      values seen by read actions within the thread.

#### 17.4.1. Shared Variables

* Memory that can be shared between threads is called shared memory or heap memory.
* All instance fields, static fields, and array elements are stored in heap memory. In this chapter, we use the term
  variable to refer to both fields and array elements.
* Local variables, formal method parameters, and exception handler parameters are never shared between threads and are
  unaffected by the memory model.
* Two accesses to (reads of or writes to) the same variable are said to be conflicting if at least one of the accesses
  is a write.

#### 17.4.2. Actions

* An inter-thread action is an action performed by one thread that can be detected or directly influenced by another
  thread.
* There are several kinds of inter-thread action that a program may perform:
    * Read (normal, or non-volatile). Reading a variable.
    * Write (normal, or non-volatile). Writing a variable.
    * Synchronization actions, which are:
        * Volatile read. A volatile read of a variable.
        * Volatile write. A volatile write of a variable.
        * Lock. Locking a monitor
        * Unlock. Unlocking a monitor.
        * The (synthetic) first and last action of a thread.
        * Actions that start a thread or detect that a thread has terminated.

* External Actions. An external action is an action that may be observable outside of an execution, and has a result
  based on an environment external to the execution.
* Thread divergence actions. A thread divergence action is only performed by a thread that is in an infinite loop in
  which no memory, synchronization, or external actions are performed. If a thread performs a thread divergence action,
  it will be followed by an infinite number of thread divergence actions.

#### 17.4.3. Programs and Program Order

* Program Order
* Among all the inter-thread actions performed by each thread t, the program order of t is a total order that reflects
  the order in which these actions would be performed according to the intra-thread semantics of t.
* A set of actions is sequentially consistent if all actions occur in a total order (the execution order) that is
  consistent with program order, and furthermore, each read r of a variable v sees the value written by the write w to v
  such that:
    * w comes before r in the execution order, and
    * there is no other write w' such that w comes before w' and w' comes before r in the execution order.

* Sequential consistency is a very strong guarantee that is made about visibility and ordering in an execution of a
  program. Within a sequentially consistent execution, there is a total order over all individual actions (such as reads
  and writes) which is consistent with the order of the program, and each individual action is atomic and is immediately
  visible to every thread.
* If a program has no data races, then all executions of the program will appear to be sequentially consistent.
* Sequential consistency and/or freedom from data races still allows errors arising from groups of operations that need
  to be perceived atomically and are not.

#### 17.4.4. Synchronization Order

#### 17.4.5. Happens-before Order

## References

* https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4.5