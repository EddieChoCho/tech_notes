# Why we need a jar/jars containing test classes

* Some classes/methods should only be use by test code but not production code.
    * e.g. Object Mother methods which can creating test data for test cases.
* To prevent developers misuse those classes/methods in the production code, we need to create jars for them.
    * Those jars could be declared as test-scope dependencies.
    * Which means that the classes/methods can only be accessible in test code not production code.

# How to create a jar containing test classes

## The easy way

* You can produce a jar which will include your test classes and resources.

```xml

<project>
    ...
    <build>
        <plugins>
            ...
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.2.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>test-jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            ...
        </plugins>
    </build>
    ...
</project>
```

* To reuse this artifact in an other project, you must declare this dependency with type test-jar :

```xml

<project>
    ...
    <dependencies>
        <dependency>
            <groupId>groupId</groupId>
            <artifactId>artifactId</artifactId>
            <classifier>tests</classifier>
            <type>test-jar</type>
            <version>version</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    ...
</project>
```

* Based on such configuration there will be two jar files generated.
    * The first one `artifactId-version.jar` contains the classes from `src/main/java`.
    * The second one `artifactId-version-tests.jar` will contain the classes from `src/test/java`.

## The preferred way

* Create a separate project:

```xml

<project>
    <groupId>groupId</groupId>
    <artifactId>artifactId-tests</artifactId>
    <version>version</version>
    ...
</project>
```

    * Move the sources files from `src/test/java` you want to share from the original project to the `src/main/java` of this project (Include the resources as well).
    * Move the required test-scoped dependencies and from the original project to this project and remove the scope (i.e. changing it to the compile-scope).

* Refer to the reusable test-classes:

```xml

<project>
    ...
    <dependencies>
        <dependency>
            <groupId>groupId</groupId>
            <artifactId>artifactId-tests</artifactId>
            <version>version</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    ...
</project>
```

## References

* [How to create a jar containing test classes](https://maven.apache.org/plugins/maven-jar-plugin/examples/create-test-jar.html)