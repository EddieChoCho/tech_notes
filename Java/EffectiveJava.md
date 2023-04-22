# Effective Java

## Ch 2. Creating and Destroying Objects

### Item 1: Consider static factory methods instead of constructors

* One advantage of static factory methods is that, unlike constructors, they have names.
* A second advantage of static factory methods is that, unlike constructors, they are not required to create a new
  object each time they’re invoked.
  * The ability of static factory methods to return the same object from repeated invocations allows classes to maintain
    strict control over what instances exist at any time.
  * Classes that do this are said to be instance-controlled.
* A third advantage of static factory methods is that, unlike constructors, they can return an object of any subtype of
  their return type.
* A fourth advantage of static factories is that the class of the returned object can vary from call to call as a
  function of the input parameters.
* A fifth advantage of static factories is that the class of the returned object need not exist when the class
  containing the method is written.

* The main limitation of providing only static factory methods is that classes without public or protected constructors
  cannot be subclassed.

### Item 2: Consider a builder when faced with many constructor parameters

* TBD

### Item 5: Prefer dependency injection to hardwiring resources

### Item 6: Avoid creating unnecessary objects

* An object can always be reused if it is immutable.
* Instead of using String.matches(), reusing the same Pattern instance.
* Prefer primitives to boxed primitives, and watch out for unintentional autoboxing.

### Item 7: Eliminate obsolete object references

* TBD
* Avoid memory leak

### Item 9: Prefer try-with-resources to try-finally

## Ch 5. Generics

### Item 26: Don’t use raw types

### Item 24: Eliminate unchecked warnings

## References

* [Effective Java, Third Edition - Keepin' it Effective](https://youtu.be/7qXfoZIqi2Q)
