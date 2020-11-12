# Maven

## Plugins

### [Maven Surefire Plugin](https://maven.apache.org/plugins-archives/maven-surefire-plugin-2.12.4/)
* The Surefire Plugin is used during the test phase of the build lifecycle to execute the unit tests of an application.

#### Running a Single Test
```
mvn -Dtest=TestCircle test
mvn -Dtest=TestCi*le test
mvn -Dtest=TestSquare,TestCi*le test
```
#### Running a set of methods in a Single Test Class
```
mvn -Dtest=TestCircle#mytest test
mvn -Dtest=TestCircle#test* test
mvn -Dtest=TestCircle#testOne+testTwo test
```

#### [log4j + mvn test](https://logging.apache.org/log4j/2.x/manual/configuration.html)


### Maven Failsafe Plugin
* The Failsafe Plugin is designed to run integration tests while the Surefire Plugin is designed to run unit tests. 
* It implies that when it fails, it does so in a safe way.
  

## Further Readings
* [What I Wish I Knew About Maven Years Ago]()

## References
* [Apache Maven Project](https://maven.apache.org/)