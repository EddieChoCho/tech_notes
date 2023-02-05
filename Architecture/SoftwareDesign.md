# SOLID

* Single Responsibility
    * A class should only have one responsibility. Furthermore, it should only have one reason to change.
    * Benefits:
        * Testing – A class with one responsibility will have far fewer test cases.
        * Lower coupling – Less functionality in a single class will have fewer dependencies.
        * Organization – Smaller, well-organized classes are easier to search than monolithic ones.

* Open for Extension, Closed for Modification

* Liskov Substitution
    * If class A is a subtype of class B, we should be able to replace B with A without disrupting the behavior of our
      program.

* Interface Segregation
    * Larger interfaces should be split into smaller ones.
    * By doing so, we can ensure that implementing classes only need to be concerned about the methods that are of
      interest to them.

* Dependency Inversion
    * Dependency Injection

# References

* [A Solid Guide to SOLID Principles](https://www.baeldung.com/solid-principles)