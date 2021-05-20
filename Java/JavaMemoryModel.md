### 17.4. Memory Model

#### 17.4.1. Shared Variables
* Memory that can be shared between threads is called shared memory or heap memory.
* All instance fields, static fields, and array elements are stored in heap memory. In this chapter, we use the term variable to refer to both fields and array elements.
* Local variables, formal method parameters, and exception handler parameters (§14.20) are never shared between threads and are unaffected by the memory model.
* Two accesses to (reads of or writes to) the same variable are said to be conflicting if at least one of the accesses is a write.

#### 17.4.5. Happens-before Order

* Two actions can be ordered by a happens-before relationship. If one action happens-before another, then the first is visible to and ordered before the second.

* If we have two actions x and y, we write hb(x, y) to indicate that x happens-before y.

    * If x and y are actions of the same thread and x comes before y in program order, then hb(x, y).
    * There is a happens-before edge from the end of a constructor of an object to the start of a finalizer for that object.
    * If an action x synchronizes-with a following action y, then we also have hb(x, y).
    * If hb(x, y) and hb(y, z), then hb(x, z).

* The wait methods of class Object have lock and unlock actions associated with them; their happens-before relationships are defined by these associated actions.

* The happens-before relation defines when data races take place.


### 17.5. final Field Semantics
* An object is considered to be completely initialized when its constructor finishes. A thread that can only see a reference to an object after that object has been completely initialized is guaranteed to see the correctly initialized values for that object's final fields.

* final Fields In The Java Memory Model
    ```
    class FinalFieldExample { 
        final int x;
        int y; 
        static FinalFieldExample f;

        public FinalFieldExample() {
            x = 3; 
            y = 4; 
        } 

        static void writer() {
            f = new FinalFieldExample();
        } 

        static void reader() {
            if (f != null) {
                int i = f.x;  // guaranteed to see 3  
                int j = f.y;  // could see 0
            } 
        } 
    }
    ```
    * The class FinalFieldExample has a final int field x and a non-final int field y. One thread might execute the method writer and another might execute the method reader.
    * Because the writer method writes f after the object's constructor finishes, the reader method will be guaranteed to see the properly initialized value for f.x: it will read the value 3. However, f.y is not final; the reader method is therefore not guaranteed to see the value 4 for it.
    
* final Fields For Security
    * One thread (which we shall refer to as thread 1) executes:
    ```
    Global.s = "/tmp/usr".substring(4);
    ```
    * while another thread (thread 2) executes:
    ```
    String myS = Global.s; 
    if (myS.equals("/tmp"))System.out.println(myS);
    ```
#### 17.5.1. Semantics of final Fields

#### 17.5.2. Reading final Fields During Construction
* A read of a final field of an object within the thread that constructs that object is ordered with respect to the initialization of that field within the constructor by the usual happens-before rules. 
* If the read occurs after the field is set in the constructor, it sees the value the final field is assigned, otherwise it sees the default value.

#### 17.5.3. Subsequent Modification of final Fields

### References
*[Chapter 17. Threads and Locks](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html)


