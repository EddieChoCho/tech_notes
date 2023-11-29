# Java 21

## Virtual Threads

### How Virtual Threads are working under the hood

* When your virtual thread detects that it is actually executing a blocking I/O operation. And instead of blocking the
  platform thread it is running on. It unmounts itself from this platform thread, moving its own context, that is its
  stack to somewhere else in the memory, namely, in the heap memory.
* The virtual thread is actually executed by a special object called a Continuation. This object is in the non-public
  part of the API, so you should not use it in production. This Continuation has a run() method that executes your
  virtual thread, and that in turn execute your task.
* When a blocking call is issued in the Java NIO API for instance, a call to Contination.yield() is made. And this is
  the call that unmounts your virtual thread from the platform thread, and copies the stack of your virtual thread to
  the heap memory.
* Of course Contination.yield() is only invoked if this blocking call is being executed in the context of a virtual
  thread of course. It works like your virtual thread was saying: "Hey, I won't be needing the CPU for a while, because
  I am waiting for a network response, so please, remove me from this platform thread".
* And by the way blocking calls are not found only in Java NIO, actually all the blocking calls you can think of in the
  JDK have been refactored to call this Continuation.yield(). And that also includes the synchronization objects from
  java.util.concurrent.
* And then at some point the data from the response is there. The operating system handler that monitors this data then
  triggers a signal, that calls continuation.run(). And this call to run() takes the stack of your virtual thread from
  the heap memory where it has been saved previously, and puts it in the wait list of the platform thread it was mounted
  on in this fork join pool.
* Because this is the way the fork join pool is working, if this thread is busy, and another thread is available, then
  this other thread will steal this task from the first thread. So you may observe that your task actually jumped from
  one platform thread to the other while it was blocked. This is how virtual threads are working under the hood.

### Handling native code and synchronized blocks: avoid pinning virtual threads

* Moving this stack around is OK, as long as your code is not playing with addresses on this stack.
* You cannot get an address on the stack in Java, but you can in C or C++. And your Java code may call some C code or
  some C++ code.
* So if the JVM detects that you have native calls in the stack of your virtual thread, it will pin it to its platform
  thread to prevent it from being moved to somewhere else.
* And a problem you may have heard before is that the synchronized block behaves in that way too.
* If you have a synchronized block in your stack, then your virtual thread is pinned on your platform thread. What can
  you do if you are in this situation?
    * Well, your first reflex should be to do nothing, and assess your code precisely.
    * A performance problem may occur if you pin a virtual thread on a platform for a long time.
    * If your native code, or your synchronized block is only doing some in-memory computations that will take in the
      order of the hundreds of nanoseconds to execute, then odds are that you will not see any performance loss.
    * It's the case if your synchronized block is guarding some in-memory data structure to ensure its correctness,
      safety and visibility.
* On the other hand, if you have some I/O operations in this synchronized block that can pin your thread for hundreds of
  milliseconds, then maybe it will be a good idea to do something about it.
* The good news is that you can replace your synchronized block with a reentrant lock, that will do the same without
  pinning your virtual thread.
* But remember, you don't have to do that, there are still many situations where pinning a virtual thread on its
  platform thread will not affect your performances.
* As usual when it comes to performances: measure, don't guess.

### Questions

* Q1. Why all Virtual Threads are daemon thread?

## ScopedValue(Preview Features)

* Scoped Values was introduced from JDK 20 as an incubating API. In JDK 21 this feature is no longer incubating;
  instead, it is a preview API.

### ThreadLocal variables drawbacks: leaking and memory footprint

* ThreadLocal has some drawbacks: potential memory leaks, and potential memory footprint overhead
  * The first one lies in the way this API has been designed.
    * Because ThreadLocal variable are in essence global mutable variables, it may be hard to check who is mutating
      them.
    * It may lead to code that is hard to follow, hard to understand and hard to debug.

  * The second one lies in the fact that a platform thread is an expensive resource in an application.
    * These threads are pooled in executors, and most of the time, they are kept alive for as long as your application
      is alive.
    * Your executor services are created when your server application is created, and are never shutdown.
    * Because a thread never dies, your ThreadLocal variables may be kept alive forever.
    * You can call remove() to remove a ThreadLocal variable from this internal hash map, but it turns out that many
      people forget to do that.
    * And that can create memory leaks in your application, which is annoying.

  * So: ThreadLocal variables are global mutable variables, that can create memory leaks.

* If you spawn new threads from one thread that has a lot of thread local variables, you may end up duplicating all of
  them, this bug will represent a significant memory footprint overhead in your application.
  * Everytime you create a new thread, this new thread checks for the current thread, the thread you are in, and by
    default, copies all the thread local variables from this current thread to the new one.
  * This is the default behavior, the one you get when you call new Thread() without any argument.
  * So if you spawn new threads from one thread that has a lot of thread local variables, you may end up duplicating all
    of them. Which may not be useful in your application. It may be a bug.
  * And this bug will represent a significant memory footprint overhead in your application.

### How Virtual Threads are changing the need for ThreadLocal

* ThreadLocal variables are fully supported by virtual threads.
* The most important thing with virtual threads is that they are cheap, so you do not need to pool them anymore.
* And when you are done with this request, instead of returning that thread to a pool for later use, you can just let it
  die.

* That solves one problem we had with ThreadLocal, which is this memory leaking that may occur if you forget to call
  remove() once you are done with a given variable.
  * Because this ThreadLocal variable will be removed when the thread dies, this problem does not exist anymore.

* But it brings another one: if you have a million virtual threads, and all of them have a copy of a bunch of
  ThreadLocal variables, then you may discover a memory footprint problem.
* If ThreadLocal variables were immutable, you could share the reference to them.

### Introducing the new requirements for ScopedValue

* First: you need a way to share some information between some components of your application, without resorting to
  method arguments, because the code you have maybe untrusted, and the information you want to share may be sensitive.
* Second: you need this way to be efficient memory-wise, and for that, sharing immutable information may be a good
  solution.
* Third: you want a bounded lifetime for this information, to avoid leaking: memory leaking, and information leaking.

### Creating and using a ScopedValue

...TBD

## References

* [Java 21 new feature: Virtual Threads #RoadTo21](https://youtu.be/5E0LU85EnTI?si=OjN6IlCphUpBR5rt)
* [Java 20 - From ThreadLocal to ScopedValue with Loom Full Tutorial - JEP Café #16](https://youtu.be/fjvGzBFmyhM?si=zSVzZhCyreysyypF)