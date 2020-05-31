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

## I/O [3]

### The File class 
* "FilePath" would have been a better name for the class. 
* It can represent either the name of a particular file or the names of a set of files in a directory.

* A directory lister
    * The File object can be used in two ways. 
        * If you call list( ) with no arguments, you’ll get the full list that the File object contains. 
        * If you want a restricted list, then you use object which implements `FilenameFilter` interface as a "directory filter," which is a class that tells how to select the File objects for display.
    
* No Directory utilities 
    * A common task in programming is to perform operations on sets of files, either in the local directory or by walking the entire directory tree.
    * We need to implement an utility class for filtering either an array of File objects in the local directory, or a List<File> of the entire directory tree starting at the given directory.

* Checking for and creating directories 
    * The `File` class is more than just a representation for an existing file or directory. 
    * You can also use a `File` object to create a new directory or an entire directory path if it does not exist. 
    * You can also look at the characteristics of files (size, last modification date, read/write), see whether a File object represents a file or a directory, and delete a file.

### Input and output
* Programming language I/O libraries often use the abstraction of a stream, which represents any data source or sink as an object capable of producing or receiving pieces of data. 
* The stream hides the details of what happens to the data inside the actual I/O device.
* `InputStream`(read()) and `OutputStream`(write()) provide valuable functionality in the form of byte-oriented I/O.
* `Reader` and `Writer` provide Unicode-compliant, character-based I/O.
* There are times when you must use classes from the "byte" hierarchy in combination with classes in the "character" hierarchy. -> `InputStreamReader`, `OutputStreamWriter`
* Try to use the `Reader` and `Writer` classes whenever you can. You’ll discover the situations when you have to use the byte-oriented libraries(`InputStream`,`OutputStream`...) because your code won’t compile.        

### Typical uses of I/O streams
* Buffered input file
    * To open a file for character input, you use a FileInputReader with a String or a File object as the file name. 
    * For speed, you’ll want that file to be buffered so you give the resulting reference to the constructor for a BufferedReader. 
    * Since BufferedReader also provides the readLine( ) method, this is your final object and the interface you read from. 
    * When readLine( ) returns null, you’re at the end of the file.

* Formatted memory input 
    * To read "formatted" data, you use a DataInputStream, which is a byteoriented I/O class (rather than char-oriented).(?) 
    * Thus you must use all InputStream classes rather than Reader classes.
    * Example - DataInputStream
        * readByte( ): Read the characters from a DataInputStream one byte at a time.
        * available( ): Find out how many more characters are available.
           
* Basic file output (BufferedWriter)
    * A `FileWriter` object writes data to a file. You’ll virtually always want to buffer the output by wrapping it in a `BufferedWriter`. 
    * Try removing this wrapping to see the impact on the performance—buffering tends to dramatically increase performance of I/O operations.
    * PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter(file))); 
    * If you don’t call close( ) for all your output files, you might discover that the buffers don’t get flushed.
    
* Text file output shortcut (PrintWriter)
    * Java SE5 added a helper constructor to PrintWriter so that you don’t have to do all the decoration by hand every time you want to create a text file and write to it.
    * PrintWriter out = new PrintWriter(file); 
    * You still get buffering, you just don’t have to do it yourself. 
    * Unfortunately, other commonly written tasks were not given shortcuts, so typical I/O will still involve a lot of redundant text.
    
* Storing and recovering data
    * writeUTF( ) and readUTF( ),  writeDouble( ) and readDouble( ), etc...
    
* Reading and writing random-access files (RandomAccessFile)
    * seek( ): move about in the file and change the values.
    * When using RandomAccessFile, you must know the layout of the file so that you can manipulate it properly.

* Piped streams
    * TBD... 
    
### File reading & writing utilities
* Reading binary files
    * TBD...
     
### Standard I/O
* Reading from standard input
    * TBD...

* Redirecting standard I/O
    * TBD...
    
### Process control
* TBD...
### New I/O
* TBD...         


## References
[1][Working With hashcode() and equals() - by Hussein Terek](https://dzone.com/articles/working-with-hashcode-and-equals-in-java)
[2][Code Review Best Practices - by Trisha Gee](https://youtu.be/a9_0UUUNt-Y)
[3] [Thinking in Java(Java SE5/6)](https://www.amazon.com/Thinking-Java-4th-Bruce-Eckel/dp/0131872486)