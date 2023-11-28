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

## References

* [Java 21 new feature: Virtual Threads #RoadTo21](https://youtu.be/5E0LU85EnTI?si=OjN6IlCphUpBR5rt)