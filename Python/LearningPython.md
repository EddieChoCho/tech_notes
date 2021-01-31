# Learning Python

# Getting Started
## How Python Runs Programs
* The Python interpreter: A program that runs the Python programs you write.
	* Depending on which flavor of Python you run, the interpreter itself may be implemented as a C program, a set of Java classes, or something else. 
	* Whatever form it takes, the Python code you write must always be run by this interpreter.

### Byte code compilation
* TBD...
* Python saves byte code with startup speed optimization process. 
* The next time you run your program, Python will load the .pyc files and skip the compilation step, as long as you haven’t changed your source code since the byte code was last saved, and aren’t running with a different Python than the one that created the byte code. 
	* Source changes: Python automatically checks the last-modified timestamps of source and byte code files to know when it must recompile—if you edit and resave your source code, byte code is automatically re-created the next time your program is run.
	* Python versions: Imports also check to see if the file must be recompiled because it was created by a different Python version, using either a “magic” version number in the byte code file itself in 3.2 and earlier, or the information present in byte code filenames in 3.2 and later.

## How You Run Programs

# Types and Operations
