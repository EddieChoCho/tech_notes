# The logback manual

## Chapter 1: Introduction

```
package chapters.introduction;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.core.util.StatusPrinter;

public class HelloWorld2 {

  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger("chapters.introduction.HelloWorld2");
    logger.debug("Hello world.");

    // print internal state
    LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
    StatusPrinter.print(lc);
  }
}
```

* Note that in the above example we have instructed logback to print its internal state by invoking the
  StatusPrinter.print() method.
  * Logback's internal status information can be very useful in diagnosing logback-related problems.

* Here is a list of the three required steps in order to enable logging in your application.
  1. Configure the logback environment. You can do so in several more or less sophisticated ways. More on this later.
  2. In every class where you wish to perform logging, retrieve a Logger instance by invoking the
     org.slf4j.LoggerFactory class' getLogger() method, passing the current class name or the class itself as a
     parameter.
  3. Use this logger instance by invoking its printing methods, namely the debug(), info(), warn() and error() methods.
     This will produce logging output on the configured appenders.

## Chapter 2: Architecture

* Logback is divided into three modules, logback-core, logback-classic and logback-access.
  * logback-core
    * The core module lays the groundwork for the other two modules.
  * logback-classic
    * The classic module extends core. The classic module corresponds to a significantly improved version of log4j.
    * Logback-classic natively implements the SLF4J API so that you can readily switch back and forth between logback
      and other logging systems such as log4j or java.util.logging (JUL) introduced in JDK 1.4.
  * logback-access
    * The access module integrates with Servlet containers to provide HTTP-access log functionality.
        
* In the remainder of this document, we will write "logback" to refer to the logback-classic module.

### Logger, Appenders and Layouts
* Logger
  * A Logger is a context for log messages.
  * This is the class that applications interact with to create log messages.

* Appender
  * Appenders place log messages in their final destinations.
  * A Logger can have more than one Appender.

* Layout
  * Layout prepares messages for outputting.
  * Logback supports the creation of custom classes for formatting messages, as well as robust configuration options for
    the existing ones.

## Chapter 3: Logback configuration

### Configuration in logback

* Logback can be configured either programmatically or with a configuration script expressed in XML or Groovy format.
* Initialization steps that logback follows to try to configure itself:
  1. Logback tries to find a file called logback-test.xml in the classpath.
  2. If no such file is found, it checks for the file logback.xml in the classpath.
  3. If no such file is found, service-provider loading facility (introduced in JDK 1.6) is used to resolve the
     implementation of com.qos.logback.classic.spi.Configurator interface by looking up the file
     META-INF\services\ch.qos.logback.classic.spi.Configurator in the class path. Its contents should specify the fully
     qualified class name of the desired Configurator implementation.
  4. If none of the above succeeds, logback configures itself automatically using the BasicConfigurator which will cause
     logging output to be directed to the console.

  * Note: For web-applications, configuration files can be placed directly under WEB-INF/classes/.

* Different configurations for test and production with Maven
  * If you are using Maven and if you place the logback-test.xml under the src/test/resources folder, Maven will ensure
    that it won't be included in the artifact produced.
  * Thus, you can use a different configuration file, namely logback-test.xml during testing, and another file, namely,
    logback.xml, in production.

* Groovy
  * Given that Groovy is a full-fledged language, we have dropped support for logback.groovy in order to protect the
    innocent.

* Fast Start-Up
  * It takes about 100 miliseconds for Joran to parse a given logback configuration file.
  * To shave off those miliseconds at aplication start up, you can use the service-provider loading facility (item 4
    above) to load your own custom Configurator class with BasicConfigrator serving as a good starting point.

### Automatically configuring logback

* The simplest way to configure logback is by letting logback fall back to its default configuration.

### Automatic configuration with logback-test.xml or logback.xml

* Example:

```xml

<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT"/>
  </root>
</configuration>
```

#### Automatic printing of status messages in case of warning or errors

* If warning or errors occur during the parsing of the configuration file, logback will automatically print its internal
  status messages on the console.
  * Note that to avoid duplication, automatic status printing is disabled if the user explicitly registers a status
    listener.

* In the absence of warnings or errors, if you still wish to inspect logback's internal status, then you can instruct
  logback to print status data by invoking the `print()` of the `StatusPrinter` class.
* The logback's internal messages, a.k.a. Status objects, which allow convenient access to logback's internal state.
* Status data
  * Enabling output of status data usually goes a long way in the diagnosis of issues with logback.
  * Note that errors can also occur post-configuration, e.g. when a disk a full or log files cannot be archived due to
    permission errors.
  * As such, it is highly recommended that you register a StatusdListener as discussed below.
  ```xml
  <configuration>
    <!-- Recommendation: place status listeners towards the the top of the configuration file -->
    <statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener" />  

    ... the rest of the configuration file  
  </configuration>
  ```
  * A StatusListner can be installed using a configuration file assuming that:
    1.the configuration file is found 2.the configuration file is well-formed XML.

  * Cases of having configuration issues:
    1. If any of these two conditions is not fulfilled, Joran cannot interpret the configuration file and in particular
       the <statusListener/> element.
    2. If the configuration file is found but is malformed, then logback will detect the error condition and
       automatically print its internal status on the console.
    3. However, if the configuration file cannot be found, logback will not automatically print its status data, since
       this is not necessarily an error condition.

    * Programmatically invoking StatusPrinter.print() ensures that status information is printed in every case.

  * Forcing Status Output
    * In the absence of status messages, tracking down a rogue logback.xml configuration file can be difficult,
      especially in production where the application source cannot be easily modified.
    * To help identify the location of a rogue configuration file, you can set a StatusListener via the "
      logback.statusListenerClass" system property to force output of status messages.

* Shorthand
  * As a shorthand, it is possible to register an OnConsoleStatusListener by setting the debug attribute to true within
    the <configuration/> element as shown below.
  ```xml
  <configuration debug="true"> 
    <!--...-->
  </configuration>
  ```
  * By the way, setting debug="true" is strictly equivalent to installing an OnConsoleStatusListener as shown
    previously.

* "logback.statusListenerClass" system property
  * One may also register a status listener by setting the "logback.statusListenerClass" Java system property to the
    name of the listener class you wish to register.
    ``` java -Dlogback.statusListenerClass=ch.qos.logback.core.status.OnConsoleStatusListener ...```
  * Note that automatic status printing (in case of errors) is disabled if any status listener is registered during
    configuration and in particular if the user specifies a status listener via the "logback.statusListenerClass"
    system.
  * Thus, by setting NopStatusListener as a status listener, you can silence internal status printing altogether
    ``` java -Dlogback.statusListenerClass=ch.qos.logback.core.status.NopStatusListener ... ```

# Logback News

## 16th of December, 2021, Release of version 1.2.9 - Prevent CVE-2021-42550 (aka LOGBACK-1591) Issue

* A successful RCE attack with CVE-2021-42550 requires all of the following conditions to be met:
  1. write access to logback.xml
  2. use of versions < 1.2.9
  3. reloading of poisoned configuration data, which implies application restart or scan="true" set prior to attack

* As an additional extra precaution, in addition to upgrading to logback version 1.2.9, we also recommend users to set
  their logback configuration files as read-only.

# References

* [The logback manual](https://logback.qos.ch/manual/index.html)
* [A Guide To Logback](https://www.baeldung.com/logback)
* [Logback News](https://logback.qos.ch/news.html)

# Further Readings

* [logbackRceDemo](https://github.com/cn-panda/logbackRceDemo)