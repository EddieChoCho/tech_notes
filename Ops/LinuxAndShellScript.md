## Navigating in the shell
* Relative paths
    * In a path, . refers to the current directory, and .. to its parent directory.
## Connecting programs
* In the shell, programs have two primary “streams” associated with them: their input stream and their output stream
* You can use `< file` and `> file` to let you rewire the input and output streams of a program.
* You can also use `>>` to append to a file.
* The `|` operator lets you “chain” programs such that the output of one is the input of another.
## Data Wrangling
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

### Regular expressions
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
* [The Missing Semester of Your CS Education](https://missing.csail.mit.edu/)