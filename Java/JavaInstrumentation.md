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

## Dynamic Load

* The procedure of loading a Java agent into an already running JVM is called dynamic load. The agent is attached using
  the [Java Attach API](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.attach/module-summary.html).
* A more complex scenario is when we already have our application running in production, and we want to add the
  instrumentation dynamically without downtime for our application.
* We'll create a AgentLoader class. For simplicity, we'll put this class in the application jar file. So our application
  jar file can both start our application, and attach our agent to the ATM application:
  ```
  VirtualMachine jvm = VirtualMachine.attach(jvmPid);
  jvm.loadAgent(agentFile.getAbsolutePath());
  jvm.detach();
  ```
* Let's also add the glue that will allow us to either start the application or load the agent. We'll call this class
  Launcher, and it will be our main jar file class:
  ```
  public class Launcher {
    public static void main(String[] args) throws Exception {
        if(args[0].equals("StartMyAtmApplication")) {
            new MyAtmApplication().run(args);
        } else if(args[0].equals("LoadAgent")) {
            new AgentLoader().run(args);
        }
    }
  }
  ```

* Starting the Application
  ```
  java -jar application.jar StartMyAtmApplication
  22:44:21.154 [main] INFO - [Application] Starting ATM application
  22:44:23.157 [main] INFO - [Application] Successful Withdrawal of [7] units!
  ```

* Attaching Java Agent
  * After the first operation, we attach the java agent to our JVM:
  ```
  java -jar application.jar LoadAgent
  22:44:27.022 [main] INFO - Attaching to target JVM with PID: 6575
  22:44:27.306 [main] INFO - Attached to target JVM and loaded Java agent successfully
  ```

* Check Application Logs
  * Now that we attached our agent to the JVM we'll see that we have the instrumentation for the second operation.
  * This means that we added our functionality on the fly, while our application was running:
    ```
    22:44:27.229 [Attach Listener] INFO - [Agent] In agentmain method
    22:44:27.230 [Attach Listener] INFO - [Agent] Transforming class MyAtm
    22:44:33.157 [main] INFO - [Application] Successful Withdrawal of [8] units!
    22:44:33.157 [main] INFO - [Application] Withdrawal operation completed in:2 seconds!
    ```

## References

* [Guide to Java Instrumentation](https://www.baeldung.com/java-instrumentation)
* [Examples](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm/src/main/java/com/baeldung/instrumentation)