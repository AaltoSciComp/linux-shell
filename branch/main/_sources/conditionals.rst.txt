Conditionals
============

Tests: ``[[ ]]``
----------------
* ``[[ expression ]]`` returns 0=true/success or 1=false/failure depending on the
  evaluation of the conditional *expression*.
* ``[[ expression ]]`` is a new upgraded variation on ``test`` (also known as ``[ ... ]``),
  all the earlier examples with single brackets that one can find online will also work
  with double
* Inside the double brackets it performs tilde expansion, parameter and variable expansion,
  arithmetic expansion, command substitution, process substitution, and quote removal
* Conditional expressions can be used to test file attributes and perform string and arithmetic
  comparisons

Selected examples file attributes and variables testing:
 - ``-f file`` true if is a file
 - ``-r file`` true if file exists and readable
 - ``-d dir`` true if is a directory
 - ``-e file`` true if file/dir/etc exists in any form
 - ``-z string`` true if the length of string is zero (always used to check that var is not empty)
 - ``-n string`` true if the length of string is non-zero
 - ``file1 -nt file2`` true if *file1* is newer (modification time)
 - many more others

::

 # check that directory does not exist before creating one
 # where $dir is a variable
 [[ -d $dir ]] || mkdir $dir

 # checks that file exists
 [[ -r $file ]] && mv $file ${file}.$(date +%Y-%m-%d) || echo $file is either non-readable or does not exist
 # in the script
 [[ -r $file ]] && mv $file ${file}.$(date +%Y-%m-%d) || { echo $file does not exist; exit 1; }
  
 # Check if script/function is given an argument
 [[ -z $1 ]] && { echo no argument; exit 1; }
 
 # Often used alternative to ${var:-a_value} or ${var:?not defined}
 [[ -n $var ]] || echo var is not defined

Note that integers have their own construction ``(( expression ))`` (we come back to this),
though ``[[ ]]`` will work for them too.  The following are more tests:

 - ``==`` strings or integers are equal  (``=`` also works)
 - ``!=`` strings or integers are not equal
 - ``string1 < string2`` true if *string1* sorts before *string2* lexicographically
 - ``>`` vice versa, for integers greater/less than
 - ``string =~ pattern`` matches the pattern against the string
 - ``&&``  logical AND, conditions can be combined
 - ``||`` logical OR
 - ``!`` negate the result of the evaluation
 - ``()`` group conditional expressions

::

 # If 'var' is ... then
 [[ $var == 'some_value' ]] && ...

 # If path is ... then 
 [[ $(pwd) == /some/path ]] && ...

 # grouping () and booleans && || within the [[ ]]
 [[ $(hostname -s) == kosh && ($(pwd) == $WORK || $(pwd) == $SCRATCH) ]] ...

 # note that [[ ]] always require spaces before and after brackets (!)


In addition (old school), double brackets inherit several operands to work with integers mainly:

 - ``-eq``, ``-ne``, ``-lt``, ``-le``, ``-gt``, ``-ge``  equal to, not equal  to,
   less  than, less than or equal to, greater than, or greater than or equal

::

 # Some use cases for [[ ]]
 
 # a popular way to check input arguments, if no input, exit (in functions
 # 'return 1').  Remember, $# is special variable for number of arguments.
 [[ $# -eq 0 ]] && { echo Usage: $0 arguments; exit 1; }
 
 # if dir exists and is not empty, then archive it
 d=path/to/dir; [[ -d $d && $(ls -A $d) ]] && tar caf $(basename $d).$(date +%Y-%m-%d).tar.gz $d
 
 # append PATH function 
 # as an exercise, we will re-implement this with the matching operator =~, see below
 appendPATH() {
   local dpath=${1:?directory is missing} && \
   [[ -d $dpath && ! $(echo $PATH|grep $dpath) ]] && export PATH+=:$dpath
 }

The matching operator ``=~`` brings more opportunities, because regular expressions come in play.
Matched strings in parentheses assigned to *${BASH_REMATCH[]}* array elements.

::

 # change shell on the Linux server if it is not BASH
 [[ ! $SHELL =~ bash ]] && chsh -s /bin/bash


* Regular expressions (regexs) are basically a mini-language for
  searching within, matching, and replacing text in strings.
* They are extremely powerful and basically required knowledge in any
  type of text processing.
* Yet there is a famous quote by Jamie Zawinski: "Some people, when
  confronted with a problem, think 'I know, I'll use regular
  expressions.' Now they have two problems."  This doesn't mean
  regular expressions shouldn't be used, but used carefully.  When
  writing regexs, start with a small pattern and slowly build it up,
  testing the matching at each phase, or else you will end up with a
  giant thing that doesn't work and you don't know why and can't debug
  it.  There are also online regex testers which help build them.
* While the basics (below) are the same, there are different forms of
  regexs!  For example, the ``grep`` program has regular regexs, but
  ``grep -E`` has extended.  The difference is mainly in the special
  characters and quoting.  Basically, check the docs for each language
  (Perl, Python, etc) you want to use regexs in.

Selected operators:

 - ``.`` matches any single character
 - ``?`` the preceding item is optional and will be matched, at most, once
 - ``*`` the preceding item will be matched zero or more times
 - ``+`` the preceding item will be matched one or more times
 - ``{N}`` the preceding item is matched exactly N times
 - ``{N,}`` the preceding item is matched N or more times
 - ``{N,M}`` the preceding item is matched at least N times, but not more than M times
 - ``[abd]``, ``[a-z]``  a character or a range of characters/integers
 - ``^``  beginning of a line
 - ``$``  the end of a line
 - ``()`` grouping items, this what comes to ${BASH_REMATCH[@]}

::

 # match an email
 email='jussi.meikalainen@aalto.fi'; regex='(.*)@(.*)'; [[ "$email" =~ $regex ]]; echo ${BASH_REMATCH[*]}

 # extract a number out of the text
 txt='A text with #1278 in it'; regex='#([0-9]+ )'; [[ "$txt" =~ $regex ]] && echo ${BASH_REMATCH[1]} || echo do not match
 
 # case insensitive matching
 var1=ABCD, var2=abcd; [[ ${var1,,} =~ ${var2,,} ]] && ...

**For case insesitive matching**, alternatively, in general, set ``shopt -s nocasematch``
(to disable it back ``shopt -u nocasematch``)


Conditionals: if/elif/else
--------------------------
Yes, we have ``[[ ]] && ... || ...`` but scripting style is more logical with if/else construction::

 if condition; then
   command1
 elif condition; then
   command2
 else
   command3
 fi

At the *condition* place can be anything what returns an exit code, i.e. ``[[ ]]``, command/function,
an arithmetic expression ``$(( ))``, or a command substitution.

::

 # to compare two input strings/integers
 if [[ "$1" == "$2" ]]
 then
   echo The strings are the same
 else
   echo The strings are different
 fi

 # checking command output
 if ping -c 1 google.com &> /dev/null; then
   echo Online
 elif ping -c 1 127.0.0.1 &> /dev/null; then
   echo Local interface is down
 else
   echo No external connection
 fi

 # check input parameters
 if [[ $# == 0 ]]; then
   echo Usage: $0 input_arg
   exit 1
 fi
 ... the rest of the code

Expanding *tarit.sh* to a script

::

 #!/bin/bash

 # usage: tarit.sh <dirname>

 dir=$1

 # if directory name is given as an argument
 if [[ -d $dir ]]; then
   archive=$(basename $dir).$(date +%Y-%m-%d).tar.gz

 # if no argument, then the current directory
 elif [[ -z $dir ]]; then
   dir='.'
   archive=$(basename $(pwd)).$(date +%Y-%m-%d).tar.gz

 # otherwise error and exit
 else
   echo $dir does not exist or empty
   exit 1
 fi

 # run tar
 tar caf $archive $dir


case
----
Another option to handle flow, instead of nested *ifs*, is ``case``.

::

 read -p "Do you want to create a directory (y/n)? " yesno   # expects user input
 case $yesno in
   y|yes)
     dir='dirname'
     echo Creating a new directory $dir
     mkdir $dir
     cd $dir
     ;;
   n|no)
     echo Proceeding in the current dir $(pwd)
     ;;
   *)
     echo Invalid response
     exit 1
     ;;
 esac
 # $yesno can be replaced with ${yesno,,} to convert to a lower case on the fly

**In the example above, we introduce** ``read``, a built-in command that reads one line from the standard
input or file descriptor.

``case`` tries to match the variable against each pattern in turn. Understands patterns rules like ``*, ?, [], |``.

Here is the *case* that could be used as an idea for your *~/.bashrc*

::

 host=$(hostname)
 case $host in
   myworkstation*)
     export PRINTER=mynearbyprinter
     # making your promt smiling when exit code is 0 :)
     PS1='$(if [[ $? == 0 ]]; then echo "\[\e[32m\]:)"; else echo "\[\e[31m\]:("; fi)\[\e[0m\] \u@\h \w $ '
   ;;
   triton*)
     [[ -n $WRKDIR ]] && alias cwd="cd $WRKDIR" && cwd
   ;;
   kosh*|brute*|force*)
     PS1='\u@\h:\w\$'
     export IGNOREEOF=0
   ;&
   *.aalto.fi)
     kinit
   ;;
   *)
     echo 'Where are you?'
   ;;
 esac

``;;`` is important, if replaced with ``;&``, execution will continue with the command
associated with the next pattern, without testing it. ``;;&`` causes the shell to test
next pattern. The default behaviour with ``;;`` is to stop matches after first pattern
has been found.

::

 # create a file 'cx'
 case "$0" in
  *cx) chmod +x "$@" ;&
  *cw) chmod +w "$@" ;;
  *c-w) chmod -w "$@" ;;
  *) echo "$0: seems that file name is somewhat different"; exit 1 ;;
 esac

 # chmod +x cx
 # ln cx cw
 # ln cx c-w
 # to make a file executable 'cx filename'

The following example is useful for Triton users: `array jobs
<https://scicomp.aalto.fi/triton/tut/array.html>`_,
where one handles array subtasks based on its index.
 

Exercise 2.3
--------------

.. exercise::

 - Re-implement the above mentioned example
   ``... [[ -d $d && ! $(echo $PATH|grep $d) ]] ...`` with the matching operator ``=~``
 - Improve the ``tarit.sh`` script we developed recently:

   - add check for the number of the given arguments. Hint: ``$#`` must be zero or one.
   - validate the given path like *path/to/file*. Hint: ``[[ $d =~ regexpr ]]``,
     the path may have only alphanumeric symbols, dots, underscore and slashes as
     a directory delimiter.

 - Expand ``cx`` script:

   - check that $@ not empty
   - add option for ``cr`` that would add read rights for all. Hint: ``chmod a+r ...``

 - (*) Write a function (add to *bin/functions*) that validates an IPv4 using
   ``=~`` matching operator. The function should fail incorrect IPs like 0.1.2.3d
   or 233.204.3.257. The problem should be solved with the regular expression only.
   Use ``return`` command to exit with the right exit code.
