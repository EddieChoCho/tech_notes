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
### Module Imports and Reloads
#### Import and Reload Basics
* In simple terms, every file of Python source code whose name ends in a .py extension is a module.
* Other files can access the items a module defines by importing that module—import operations essentially load another file and grant access to that file’s contents.
* Import(statement)
    * Import operations run the code in a file that is being loaded as a final step.
        * Importing a file is yet another way to launch it.
        * After the first import, later imports do nothing, even if you change and save the module’s source file again in another window.
    * Imports must find files, compile them to byte code, and run the code. Which are too expensive an operation to repeat more than once per file, per program run.

* Reload(function)
    * If you really want to force Python to run the file again in the same session without stopping and restarting the session, you need to instead call the reload function available in the imp standard library module.
    ```
    >>> from imp import reload # Must load from module in 3.X (only)
    >>> reload(script1)
    ```
    * This allows you to edit and pick up new code on the fly within the current Python interactive session.

# Types and Operations
