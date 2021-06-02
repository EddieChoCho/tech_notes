# New Features in Java 9

### Modular System
* Modules have a concept of dependencies, can export a public API and keep implementation details hidden/private.
* One of the main motivations here is to provide modular JVM, which can run on devices with a lot less available memory. The JVM could run with only those modules and APIs which are required by the application. 
* Simply put, the modules are going to be described in a file called module-info.java located in the top of java code hierarchy:
    ```
    module com.baeldung.java9.modules.car {
        requires com.baeldung.java9.modules.engines;
        exports com.baeldung.java9.modules.car.handling;
    }
    * Our module car requires module engine to run and exports a package for handling.

### A New HTTP Client
* A long-awaited replacement of the old HttpURLConnection. The new API is located under the jdk.incubator.http.
* It should support both HTTP/2 protocol and WebSocket handshake, with performance that should be comparable with the Apache HttpClient, Netty and Jetty.
* Example:
    ```
    HttpRequest request = HttpRequest.newBuilder()
        .uri(new URI("https://postman-echo.com/get"))
        .GET()
        .build();

    HttpResponse<String> response = HttpClient.newHttpClient()
        .send(request, HttpResponse.BodyHandler.asString());

    ```
### Process API
### Small Language Modifications
#### Try-With-Resources
* In Java 7, the try-with-resources syntax requires a fresh variable to be declared for each resource being managed by the statement
* In Java 9 there is an additional refinement for : if the resource is referenced by a final or effectively final variable, a try-with-resources statement can manage a resource without a new variable being declared:
    ```
    MyAutoCloseable mac = new MyAutoCloseable();
    try (mac) {
        // do some stuff with mac
    }
 
    try (new MyAutoCloseable() { }.finalWrapper.finalCloseable) {
        // do some stuff with finalCloseable
    } catch (Exception ex) { }
    ```
#### Diamond Operator Extension
* Now we can use diamond operator in conjunction with anonymous inner classes.

#### Interface Private Method

### Shell Command Line Tool

### JCMD Sub-Commands

### Мulti-Resolution Image API

### Variable Handles

### Publish-Subscribe Framework

### Unified JVM Logging

### New APIs

#### Immutable Set
* java.util.Set.of() – creates an immutable set of a given elements
* The Set returned by this method is JVM internal class: java.util.ImmutableCollections.SetN, which extends public java.util.AbstractSet. 
* If we try to add or remove elements, an UnsupportedOperationException will be thrown.

#### Optional to Stream
* java.util.Optional.stream() gives us an easy way to you use the power of Streams on Optional elements

## References
* [New Features in Java 9](https://www.baeldung.com/new-java-9)
