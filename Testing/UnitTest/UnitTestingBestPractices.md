## Unit Testing Best Practices

### Readability 
* Make sure setup and teardown methods are not abused. It’s better to use factory methods for readability.
* Make sure the test tests one thing only.
* Check for good and consistent naming conventions.
* Make sure that only meaningful assert messages are used, or none at all (meaningful test names are better).
* Make sure tests don’t use magic strings and values as inputs. use the simplest inputs possible to prove your point.
* Make sure there is consistency in location of tests. make it easy to find related tests for a method, or a class, or a project. 

### Maintainability
* Make sure tests are isolated from each other and repeatable.
* Make sure that testing private or protected methods is not the norm (public is always better).
* Make sure strict mocks are used as little as possible (leads to over specification and fragile tests). 
 
### Trust
* Make sure the test does not contain logic or dynamic values.
* Check coverage by playing with values (booleans or consts).
* Make sure unit tests are separated from integration tests.
* Make sure tests don’t use things that keep changing in a unit test (like DateTime.Now ). Use fixed values.
* Make sure tests don’t assert with expected values that are created dynamically - you might be repeating production code.


## References
* [The Art of Unit Testing](https://www.manning.com/books/the-art-of-unit-testing-second-edition)
* [Unit Testing Best Practices with Roy Osherove](https://youtu.be/dJUVNFxrK_4)
* [Unit Testing Review Guidelines by Roy Osherove](https://www.artofunittesting.com/unit-testing-review-guidelines/)