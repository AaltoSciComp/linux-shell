Find
====

* ``find`` is a very unixy program: it finds files, but in the most
  flexible way possible.
* It is a amazingly complicated program
* It is a number one in searching files in shell

With no options, just recursively lists all files starting in current directory::

  find

The first option gives a starting directory::

  find /etc/

Other search options: by modification/accessing time, by ownership, by access
type, joint conditions, case-insensitive, that do not match, etc [#find1]_
[#find2]_::

 # -or-  'find ~ $WRKDIR -name file.txt' one can search more than one dir at once
 find ~ -name file.txt
 
 # look for jpeg files in the current dir only
 find . -maxdepth 1 -name '*.jpg' -type f
 
 # find all files of size more than 10M and less than 100M
 find . -type -f -size +10M -size -100M
 
 # find everything that does not belong to you
 find ~ ! -user $USER | xargs ls -ld
 
 # open all directories to group members
 # tip: chmod applies x-bit to directories automatically
 find . -type d -exec chmod g+rw {} \;
 
 # find all s-bitted executable binaries
 find /usr/{bin,sbin} -type f -perm -u+x,u+s
 
 # find and remove all files older than 7 days
 find path/dir -type f -mtime +7 -exec rm -f {} \;

Find syntax is actually an entire boolean logic language given on the
command line: it is a single expression evaluated left to right with
certain precedence.  There are match expressions and action
expressions.  Thus, you can get amazingly complex if you want to.
Take a look at the 'EXAMPLES' section in *man find* for the comprehensive list
of examples and explanations.

**find on Triton**  On Triton's WRKDIR you should use ``lfs find``.  This uses a raw lustre connection
to make it more efficient than accessing every file. It has somewhat limited abilities as comparing
to GNU find. For details ``man lfs`` on Triton.

**Fast find -- locate**  Another utility that you may find useful ``locate <pattern>``, but on
workstations only.  This uses a cached database of all files, and
just searches that database so it is much faster.

**Too many arguments**  error solved with the ``find ... | xargs``


.. [#find1] https://alvinalexander.com/unix/edu/examples/find.shtml
.. [#find2] http://www.softpanorama.org/Tools/Find/index.shtml
