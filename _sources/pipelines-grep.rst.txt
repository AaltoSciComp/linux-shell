Evaluations, separators and grep
================================

Evaluations and separators:  ;  &&  ||
--------------------------------------

- With ; you can put several commands on the same line.

Chaining: ``command_a ; command_b``: always runs both commands.

Remember exit codes?  In shell, 0=success and anything 1-255=failure.
Note that this is opposite of normal Boolean logic!

The ``&&`` and ``||`` are `short-circuit
<https://en.wikipedia.org/wiki/Short-circuit_evaluation>`__ (lazy)
boolean operators.  They can be used for quick conditionals.

* ``command_a && command_b``

  * If ``command_a`` is successful, also run ``command_b``
  * final exit code is last evaluated one, which has the role of Boolean *and*.

* ``command_a || command_b``

  * If ``command_a`` is *not* successful, also run ``command_b``
  * final exit code is that of the last evaluated command, which has
    the role of Boolean *or*.

**Hint** ``command_a && command_b || command_c``

Try: ``cd /nonexistent_dir && ls /nonexistent_dir`` compare with ``cd /nonexistent_dir; ls /nonexistent_dir``

Try: ``ping -c 1 8.8.8.8 > /dev/null && echo online || echo offline``


grep
----
Later on you'll find out that ``grep`` is one of the most useful
commands you ever discover on Linux (except for all the *other* most
useful commands ever)

::

 grep <pattern> <filename>  # grep lines that match <pattern>
  -or- 
 command | grep <pattern>  # grep lines from stdin

::

 # search all the files in the dir/ and its subdirs, to match the word 'is', case insensitive
 grep -R -iw 'is' dir/
 
 # grep all lines from *command* output, except those that have 'comment' in it
 *command* | grep -v comment
 
 # displaying 2 extra lines before and after the match (-A just after, -B just before)
 grep -C 2 'search word' file
 
 # counts the number of matches
 grep -c <pattern> file(s)
 
 # shows only the matched part of the string (by default grep shows whole line)
 grep -o <pattern> file(s)
 
 # accepts way more advanced regular expressions as a search pattern
 grep -E <extended_regexpr> file(s)

For details on what <pattern> could be, look for REGULAR EXPRESSIONS
at ``man grep``.  Some examples:

::

 # grep emails to a list
 grep -Eio "\b[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,6}\b" file.txt
 
 # grep currently running firefox processes
 ps auxw | grep firefox
 
 # grep H1 and H2 header lines out of HTML file
 grep "<[Hh][12]>" file.html


Exercise 1.4
------------

.. exercise::

 - make a pipe that counts number of files/directories (including dot files) in your directory
 - grep directories out of ``ls -l``
 - grep all but blank lines of the 'man cut | grep ...'
 - Using pipes and commands echo/tr/uniq, find doubled words out of ``My
   Do Do list: Find a a Doubled Word``.
 - If you are on a multiuser system, count unique logged in users.
   Tip: ``w`` or ``users`` gives you a list of all currently login users,
   many of them have several sessions open. Commands to discover: cut / sort / wc
 - (*) Play with the commands grep, cut: find at least two ways to
   extract IP addresses out of /etc/hosts. Tip: *grep* has *-o* option, thus one can build
   a regular expression that will grab exactly what you need.
