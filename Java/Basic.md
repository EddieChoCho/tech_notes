## equals and hashcode

### Default Implementations
* equals(Object obj): a method provided by java.lang.Object. The default implementation is: "two objects are equal if and only if they are stored in the same memory address."
* hashcode(): a method provided by java.lang.Object that returns an integer representation of the object memory address. By default, this method returns a random integer that is unique for each instance. This integer might change between several executions of the application and won't stay the same.

### The Contract Between equals() and hashcode()
* If two objects are equal, they MUST have the same hash code.
* If two objects have the same hash code, it doesn't mean that they are equal.
* Overriding equals() alone will make your business fail with hashing data structures like: HashSet, HashMap, HashTable ... etc.
* Overriding hashcode() alone doesn't force Java to ignore memory addresses when comparing two objects.
* If the equals() is needed to be overridden, then we also need to override the hashcode() method.

## References
[1][Working With hashcode() and equals() - by Hussein Terek](https://dzone.com/articles/working-with-hashcode-and-equals-in-java)
[2][Code Review Best Practices - by Trisha Gee](https://youtu.be/a9_0UUUNt-Y)