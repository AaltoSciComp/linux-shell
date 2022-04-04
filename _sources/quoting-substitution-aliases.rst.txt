Quoting, substitutions, aliases
===============================

Last time, we focused on interactive things from the command line.
Now, we build on that some and end up with making our own scripts.

Command line processing and quoting
-----------------------------------
So, shell is responsible for interpreting the commands you type. Executing commands
might seem simple enough, but a lot happens between the time you press RETURN and
time your computer actually does something.

* When you enter a command line, it is one string.
* When a program runs, it always takes an array of strings (the
  ``argv`` in C, ``sys.argv`` in Python, for example).  How do you get
  from one string to an array of strings?  Bash does a lot of
  processing.
* The simplest way of looking at it is everything separated by spaces,
  but actually there is more: variable substitution, command
  substitution, arithmetic evaluation, history evaluation, etc.

The partial order of operations is (don't worry about exact order:
just realize that the shell does a lot of different things in same
particular order):

* history expansion
* brace expansion (``{1..9}``)
* parameter and variable expansion (``$VAR``, ``${VAR}``)
* command substitution (``$()``)
* arithmetic expansion (``$((1+1))``)
* word splitting
* pathname expansion (``*``, ``?``, ``[a,b]``)
* redirects and pipes

One thing we will start to see is shell quoting.  There are several types
of quoting (we will learn details of variables later)::

  # Double quotes: disable all other characters except $, ', \  
  echo "$SHELL"
  
  # Single quotes: disable all special characters
  echo '$SHELL'
  
  # backslash disables the special meaning of the next character
  ls name\ with\ space

By special characters we mean::

 # & * ? [ ] ( ) { } = | ^ ; < > ` $ " ' \

There are different rules for embedding quoting in other quoting.
Sometimes a command passes through multiple layers and you need to
really be careful with multiple layers of quoting!  This is advanced,
but just remember it.

::

 echo 'What's up? how much did you get $$?'      # wrong, ' can not be in between ''
 echo "What's up? how much did you get $$?"      # wrong, $$ is a variable in this case
 echo "What's up? how much did you get \$\$?"    # correct
 echo "What's up? how much did you get "'$$'"?"  # correct

At the end of the line ``\`` removes the new line character, thus the command can continue to a next line::

 ping -c 1 8.8.8.8 > /dev/null && \
 echo online || \
 echo offline

.. _linux-training-substitute-command-output:

Substitute a command output
---------------------------
* Command substitutions execute a command, take its stdout, and  place
  it on the command line in that place.

``$(command)`` or alternatively ```command```. Could be a command or a
list of commands with pipes, redirections, grouping, variables
inside. The ``$()`` is a modern way, supports nesting, works inside double
quotes.  To understand what is going on in these, run the inner
command first.

::

 # 'whoami' alternative
 echo $(id -un):$(id -gn)@$(hostname -s) 
 
 # save current date to a variable
 today=$(date +%Y-%m-%d)
 
 # create a new file with current timestamp in the name (almost unique filename)
 touch file.$(date +%Y-%m-%d-%H-%M-%S)
 
 # archive current directory content, where new archive name is based on current path and date
 tar czf $(basename $(pwd)).$(date +%Y-%m-%d).tar.gz .
 
This is what makes BASH powerful!

Note:  ``$(command || exit 1)`` will not have an effect you expect, command is executed in a
subshell, exiting from inside a subshell, closes the subshell only not the parent script. 
Subshell can not modify its parent shell environment, though can give back exit code or signal it::

 # this will not work, echo still will be executed
 dir=nonexistent
 echo $(ls -l $dir || exit 1)
 
 # this will not work either, since || evaluates echo's exit code, not ls
 echo $(ls -l $dir) || exit 1
 
 # this will work, since assignment a comman substitution to a var returns exit
 # code of the executed command
 var=$(ls -l $dir) || exit 1
 echo $var


More about redirection, piping and process substitution
-------------------------------------------------------
*STDIN*, *STDOUT* and *STDERR*: reserved file descriptors *0*, *1* and *2*. They always there
whatever process you run. But one can use other file descriptors as well.

*File descriptor* is a number that uniquely identifies an open file.

*/dev/null*  file (actually special operating system device) that
discards all data written to it.

::

 # discards STDOUT only
 command > /dev/null
 
 # discards both STDOUT and STDERR
 command &> /dev/null
 command > /dev/null 2>&1    # same as above, old style notation
 
 # redirects outputs to different files
 command 1>file.out 2>file.err
 
 # takes STDIN as an input and outputs STDOUT/STDERR to a file
 command < input_file &> output_file

Note, that ``&>`` and ``>&`` will do the same, redirect both STDOUT and STDERR
to the same place, but the former syntax is preferable.

::

 # what happens if 8.8.8.8 is down? How to make the command more robust?
 ping -c 1 8.8.8.8 > /dev/null && echo online || echo down
 
 # takes a snapshot of the directory list and send it to email, then renames the file
 ls -l > listing && { mail -s "ls -l $(pwd)" jussi.meikalainen@aalto.fi < listing; mv listing listing.$(date +"%Y-%m-%d-%H-%M"); }
 
 # a few ways to empty a file
 > filename
 cat /dev/null > filename
 
 # read file to a variable
 var=$(< path/to/file)
 
 # extreme case, if you can't get the program to stop writing to the file...
 ln -s /dev/null filename
 
Pipes are following the same rules with respect to standard output/error. In order to pipe both STDERR and STDOUT ``|&``.

If ``!``  preceeds the command, the exit status is the logical negation.

**tee** in case you still want output to a terminal and to a file ``command | tee filename``

``exec > output.txt`` or ``exec 2> errors.txt`` executed in the script will send the output to 
the file, standard output or error output correspondingly. Opening other than standard file
descriptors: **exec** causes the shell to hold the file descriptor until the shell dies or
closes it.

::

 # open input_file for reading into the file descriptor 3
 exec 3< $input_file
 # while open, any command can operate on the descriptor
 read -n 3 var <&3
 command <&3
 # mind the file offset, one can read a line, or a few chars, if you have read the file
 # to the end, to reset the offset, run another 'exec 3< ...'
 # close the descriptor after you are done
 exec 3>&-

 # similar for writing
 exec 5> $output_file; command > &5; ...; exec 5>&-
 # or appending (keep in mind that you use >> only to open the file)
 exec 5>> $output_file; command > &5; ...; exec 5>&-
 # or writing and reading
 exec 6<>$file; ... exec 6<>&-
 # or use a name instead of the descriptor numeric value
 exec {out}>$output_file; ... echo something >&$out; ...
 # redirecting descriptor to another one
 exec 3>&1

Opening a FD instead of using a file name multiple times may save you some IO. *Hint:* to monitor the
file operations (system calls) one may employ **strace -f -c -e trace=write,openat your_script**.

But what if you need to pass to another program results of two commands at once? Or if command
accepts file as an argument but not STDIN?

One can always do this in two steps, run commands and save results to file(s) and then use
them with the another command. Though BASH helps to make even this part easier (or harder),
the feature called
*Process Substitution*, looks like ``<(command)`` or ``>(command)``, no spaces in between
parentheses and < signs. It emulates a file creation out of *command* output
and place it on a command line. The *command* can be a pipe, pipeline etc.

The actual file paths substituted are */dev/fd/<n>*. The file paths can be passed as an
argument to the another command or just redirected as usual.

::

 # BASH creates a file that has an output of *command2* and pass it to *command1*
 # file descriptor is passed as an argument, assuming command1 can handle it
 command1 <(command2)
 
 # same but redirected (like: cat < filename)
 command1 < <(command2)
 
 # in the same way one can substitute results of several commands or command groups
 command1 <(command2) <(command3 | command4; command5)
 
 # example: comparing listings of two directories
 diff <(ls dir1) <(ls dir2)
 
 # and vice versa, *command1* output is redirected as a file to *command2*
 command1 > >(command2)
 
 # essentially, in some cases pipe and process substituion do the same
 ls -s | cat
 cat <(ls -s)


Aliases
-------
* Alias is nothing more than a shortcut to a long command sequence
* With alias one can redefine an existing command or name a new one
* Alias will be evaluated only when executed, thus it may have all the expansions and
  substitutions one normally has on the cli
* They are less flexible than functions which we will discuss next

::

 # your own listing command
 alias l='ls -lAF'
 
 # shortcut for checking space usage
 alias space='du -hs .[!.]* * | sort -h'
 
 # prints in the compact way login:group
 alias me='echo "$(id -un):$(id -gn)"'
 
 # redefine rm
 alias rm='rm -i'
 alias rm='rm -rf'

Aliases go to *.bashrc* and available later by default (really,
anywhere they can be read by the shell).



Exercise 2.1
------------

[Lecturer's notes: about 40 mins joint hands-on session + break]

.. exercise::

 - Use command substitution to create an empty file with the date in the name, like
   ``file.YYYY-MM-DD.out``. Tip: investigate ``date +"..."`` examples above and/or ``man date``.
 - Learn Brace expansions ``echo {0..9} {a..z}``. Using it, create five directories (``mkdir``) in
   the current folder with the names like: DIR.NUMBER.CURRENT_YEAR, example mydir.1.2022, mydir.2.2022
 - Make a command (so called one-liner) with ``ls``, ``echo``, redirections etc that takes a file path
   and says whether this file/directory exists or not. Redirect STDOUT/STDERR to /dev/null.
   Take ``ping -c 8.8.8.8 ...`` as an example.
 - Use the example in the text above to send ``du -hs * .[!.]* | sort -h`` output to yourself via email.
 - (*) Use any of the earlier created files to compare there modification times with ``stat -c '%y' filename``,
   ``diff`` and the process substitution. 
 - (*) Using pipes and commands ``echo``, ``tr``, ``uniq``, find doubled words out of
   ``My Do Do list: Find a a Doubled Word.``
 - (*) Join *find* and *grep* power and find all the files in /{usr/,}{bin,sbin} that have '#!/bin/bash' in it
