## Methods for Executing a Shell Script[1]
### Method 1 - Use the filename to run the shell script
* The most common, as well as the simplest and most straight forward, method to execute shell scripts.
* This method requires us to have the shebang line in our script. 
* We simply use the absolute pathname or the relative pathname to specify the shell script to execute.
* Changing Permissions to Make a Shell Script Executable
    * e.g. sets execute permission for user and group. ```$ chmod +x testscript```
### Method 2 - Specify the interpreter as a way to execute the shell script
* We can also execute a shell script by specifying the interpreter on the command line.
```sbtshell
$ bash myscript
Hello from myscript
```
* This tells "exec" to use the bash shell to run the myscript file. 
* Specifying the interpreter on the command line forces Linux to use that shell to run the script, no matter what is specified on the shebang line inside of the script.
* Since we are specifying which shell interpreter will be used on the command line THIS is an instance where we do not have to include a shebang line in the file. 
* However, it is a wise habit to include the shebang line, including the appropriate shell interpreter in your file to avoid any confusion by someone else who may be running or making updates to your script.

### Method 3 - Using . ./ (dot space dot slash) to execute the shell script
* Using this method will execute the shell script in the current shell without forking a new shell instance to run the script.
* Note: What does Forking / Fork do?
    * In Linux the term "fork" means that a running process creates a new child process which is identical to its parent except that it has a different system process id. 
    * When the child process completes then control is returned to the parent process - which may be the command line interpreter.
* This method (and the source command below) actually runs the shell script in the same process id as the current command line shell. What is the advantage of this?
    * There are two files that control your login environment .bashrc or .bash_profile.
    * If you edit one or both of these files the changes do NOT take effect until the files are executed. 
    * if you simply run them as a shell script the new shell that is created will have the new settings, but NOT your login shell. 
    * After the changes are made we can either logout and login back in to allow the changes to take effect, or we can use the "dot space dot slash" method to execute the altered file for the changes to take place.


### Method 4 - Using the source command to execute the shell script
* The second most used method to run a shell script.
* This is the same as the .(dot) method explained above.

## Executing a shell script from another script[1]
```sbtshell
$ cat myscript
#! /usr/bin/env bash
echo "Hello from myscript"
# The following line calls another script found in my library
two_script
echo "We are done"
```
```sbtshell
$ cat two_script
#! /usr/bin/env bash
echo "This is two_script"
exit 1
```
```sbtshell
$ ./myscript
Hello from myscript
This is two_script
We are done
```
* Executing a shell script from another script with `source`:
```sbtshell
$ cat myscript
#! /usr/bin/env bash
echo "Hello from myscript"
# The following line calls another script found in my library
source two_script
echo "We are done"

$ ./myscript
Hello from myscript
This is two_script
```
    * The source command - that it actually runs the shell script in the same shell as the calling script.
    * If we do not spawn a separate shell, which is the case with the source command, then when two_script executes the exit statement it exits the original shell. 
    * In the first instance two_script actually spawns a separate shell to run the two_script in and it is that shell that ends when the exit command is run. 

## Navigating in the shell[2]
* Relative paths
    * In a path, . refers to the current directory, and .. to its parent directory.
## Connecting programs[2]
* In the shell, programs have two primary “streams” associated with them: their input stream and their output stream
* You can use `< file` and `> file` to let you rewire the input and output streams of a program.
* You can also use `>>` to append to a file.
* The `|` operator lets you “chain” programs such that the output of one is the input of another.
## Data Wrangling[2]
```sbtshell
$ ssh myserver journalctl

# limit logs to ssh stuff:
$ ssh myserver journalctl | grep sshd

# Additional quoting: It’s wasteful to stream it all to our computer and then do the filtering. Instead, we can do the filtering on the remote server, and then massage the data locally. 
# `less` gives us a “pager” that allows us to scroll up and down through the long output. 
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less

# To save some additional traffic while we debug our command-line, we can even stick the current filtered logs into a file so that we don’t have to access the network while developing.
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$ less ssh.log

```
* Stream editor - sed
```sbtshell
# The s command is written on the form: s/REGEX/SUBSTITUTION/, where REGEX is the regular expression you want to search for, and SUBSTITUTION is the text you want to substitute matching text with.
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

### Regular expressions[2]
*  Regular expressions are usually (though not always) surrounded by `/`.
* Common patterns:
    * `.` means “any single character” except newline
    * `*` zero or more of the preceding match
    * `+` one or more of the preceding match
    * `[abc]` any one character of a, b, and c
    * `(RX1|RX2)` either something that matches RX1 or RX2
    * `^` the start of the line
    * `$` the end of the line


### Cron Job
```sbtshell
# Edit cron jobs
$ crontab -e

# min hour day month week
$ * * * * * date > ~/cron_job.log

# Show all cron jobs 
$ crontab -l

# Remove cron jobs
$ crontab -r 

```

## References
* [1][4 Methods for Running a Shell Script on Linux or UNIX with Examples...and BONUS debugging techniques!!! - Part I](https://www.livefirelabs.com/unix_tip_trick_shell_script/unix_shell_scripting/4-methods-for-running-a-shell-script-on-linux-or-unix-with-examples-and-bonus-debugging-techniques-part-1.htm)
* [2][The Missing Semester of Your CS Education](https://missing.csail.mit.edu/)