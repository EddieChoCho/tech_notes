## Mocking

### 10 Capturing invocation arguments for verification
* Invocation arguments can be captured for later verification through a set of special "withCapture(...)" methods. There are three different cases, each with its own specific capturing method:
    * verification of arguments passed to a mocked method, in a single invocation: T withCapture();
    * verification of arguments passed to a mocked method, in multiple invocations: T withCapture(List<T>);
    * verification of arguments passed to a mocked constructor: List<T> withCapture(T).

* 10.1 Capturing arguments from a single invocation
    ```
    new Verifications() {{
        double d;
        String s;
        mock.doSomething(d = withCapture(), null, s = withCapture());

        assertTrue(d > 0.0);
        assertTrue(s.length() > 1);
    }};
    ```
    * The withCapture() method can only be used in verification blocks.

* 10.2 Capturing arguments from multiple invocations
    ```
    new Verifications() {{
        List<DataObject> dataObjects = new ArrayList<>();
        mock.doSomething(withCapture(dataObjects));
    
        assertEquals(2, dataObjects.size());
        DataObject data1 = dataObjects.get(0);
        DataObject data2 = dataObjects.get(1);
        // Perform arbitrary assertions on data1 and data2.
    }};

    ```
    * Differently from withCapture(), the withCapture(List) overload can also be used in expectation recording blocks.
   
* 10.3 Capturing new instances
    ```
    new Verifications() {{
        // Captures the new instances created with a specific constructor.
        List<Person> personsInstantiated = withCapture(new Person(anyString, anyInt));
    
        // Now captures the instances of the same type passed to a method.
        List<Person> personsCreated = new ArrayList<>();
        dao.create(withCapture(personsCreated));
    
        // Finally, verifies both lists are the same.
        assertEquals(personsInstantiated, personsCreated);
    }};

    ```




## References
* [Tutorial - JMockit](https://jmockit.github.io/tutorial.html)