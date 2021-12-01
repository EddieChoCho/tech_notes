# Java Instrumentation

## What Is a Java Agent

* In general, a java agent is just a specially crafted jar file. It utilizes
  the [Instrumentation API](https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/Instrumentation.html)
  that the JVM provides to alter existing byte-code that is loaded in a JVM.
* For an agent to work, we need to define two methods:
    * premain – will statically load the agent using -javaagent parameter at JVM startup
    * agentmain – will dynamically load the agent into the JVM using the Java Attach API

* Note: A JVM implementation, like Oracle, OpenJDK, and others, can provide a mechanism to start agents dynamically, but
  it is not a requirement.

## Loading a Java Agent

* To be able to use the Java agent, we must first load it.
* We have two types of load:
    * static – makes use of the premain to load the agent using -javaagent option
    * dynamic – makes use of the agentmain to load the agent into the JVM using the Java Attach API

### Static Load

* Loading a Java agent at application startup is called static load.
* `Static load modifies the byte-code at startup time before any code is executed`.
* Static load uses the premain method, which will run before any application code runs.
* Execute:
    ```
    java -javaagent:agent.jar -jar application.jar
    ```
    * Note: we should always put the –javaagent parameter before the –jar parameter.

* Logs:
    ```
    22:24:39.296 [main] INFO - [Agent] In premain method
    //premain method ran
    22:24:39.300 [main] INFO - [Agent] Transforming class MyAtm
    //MyAtm class was transformed
    22:24:39.407 [main] INFO - [Application] Starting ATM application
    22:24:41.409 [main] INFO - [Application] Successful Withdrawal of [7] units!
    22:24:41.410 [main] INFO - [Application] Withdrawal operation completed in:2 seconds!
    //the time of completion for a transaction was added by the Java agent
    22:24:53.411 [main] INFO - [Application] Successful Withdrawal of [8] units!
    22:24:53.411 [main] INFO - [Application] Withdrawal operation completed in:2 seconds!
    //the time of completion for a transaction was added by the Java agent
    ```

## References

* [Guide to Java Instrumentation](https://www.baeldung.com/java-instrumentation)