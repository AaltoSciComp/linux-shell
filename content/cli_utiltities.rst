Command line utilities
======================

Utilities: the building blocks of shell
---------------------------------------

 - wide range of all kind of utilities available in Linux
 - shell is a glue to bind them all together
 - commandline is often a long list of those utilities joint into pipe
   that pass output of each other further

::

 echo, pwd, id, hostname, uname, ps, top, pstree, bg/fg, jobs, kill, touch
 ls, cd, cp, rm, mv, mkdir, ln, type, stat, file, du, chmod, chgrp (chown),
 find, tar, gzip, sftp, rsync, man, nano (vim/emacs), less, ssh, ...
 grep, cat, tr, cut, sort, head, tail, uniq, col, xargs,
 date, wc, cal, nl, diff, alias, df, basename, w, split, tee, 
 sed, awk, paste, ...

Additional utilities for the software development, system administration etc

**Coreutils by GNU** You may find many other useful commands at
https://www.gnu.org/software/coreutils/manual/coreutils.html


Input and output: redirect and pipes
------------------------------------

* stdout and stdin from the processes section, you remember right? each process has it
* by default *stdout* goes to the screen and *stdin* expects input from the keyboard
* we can change that on demand: pipes and redirections

Pipe: output of the first command as an input for the second one ``command_a | command_b``::

 # see what files/directories use the most space, including hidden ones
 du -hs * .[!.]* | sort -h | tail -n 10
 
 # count a number of logged in users
 w -h | wc -l

 # send man page to a default printer
 man -t ls | lpr

 # print all non-printable characters as well
 ls -lA | cat -A
 
Redirects:
 - Like pipes, but send data to/from files instead of other processes.
 - Replace a file: ``command > file.txt``
 - Append to a file: ``command >> file.txt`` (be careful you do not mix them up!)
 - Redirect file as STDIN: ``command < file``  (in case program accepts STDIN only)

::

 echo Hello World > hello.txt
 
 ls -lH >> current_dir_ls.txt
 
 # create a few dummy files
 echo 'a b c' > file1
 echo 'x y z' > file2

 # join two files into one
 cat file1 file2 > file3

 # go through file1 and replace spaces with a new line mark, then output to file2
 tr -s ' ' '\n' < file1 > file4
 # the same result but another approach: (and more readable format)
 cat file2 | tr -s ' ' '\n' > file5
  
 # join file1 and 2 lines one by one using : as a delimiter
 paste -d : file4 file5 > file6
 
 # get rid of output, 'null' is a special device
 command > /dev/null

**This is the unix philosophy** and the true power of the shell.  The
unix philosophy is a lot of small, specialized, good programs
which can be easily connected together. The beauty of the cli are elegant one-liners
i.e. list of commands executed in one line.::

 # tar and copy directory to a remote host
 tar czf - path/to/dir | ssh LOGIN@remote.server.fi 'cat > path/to/archive/dir/archive_file.tar.gz'
 
 # to remove all carriage returns and Ctrl-z characters from a Windows file
 cat win.txt | tr -d '\15\32' > unix.txt

 # extract user names and store them to a file
 getent passwd | cut -d: -f1,5 > users

 # print the name of the newest file in the directory (non-dot)
 ls -1tF | grep -v -E '*/|@' | head -1

http://www.bashoneliners.com/


Groupping commands
------------------

To dump output of all commands at once: group them.

::

 { command1; command2; } > filename  # commands run in the current shell  as a group
 ( command1; command2; ) > filename  # commands run in external shell as a group

stderr
------

A separate stream (=file descriptor), though we can direct it as well::

 # redirect both stderr and stdout to /dev/null
 ping -c 1 8.8.8.8 &> /dev/null
 
 # piping both
 command_a &| command_b

Advanced usage cases, like subshells, process substitution, other file descriptors
than stdin/stderr/stdout etc will be covered in the Part 2 of this course.
