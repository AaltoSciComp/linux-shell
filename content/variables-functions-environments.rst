Variables, functions, environment
=================================

Your ~/bin and PATH
-------------------
The PATH is an environment variable. It is a colon delimited list of directories that your
shell searches through when you enter a command. Binaries are at */bin*, */usr/bin*,
*/usr/local/bin* etc. The best place for your own is *~/bin*.::

 # add to .bashrc
 export PATH="$PATH:$HOME/bin"
 # after you have your script written, set +x bit and run it
 chmod +x ~/bin/script_name.sh
 script_name.sh

You can find where a program is using ``which`` or ``type -a``, we recommend the later one::

  type -a ls      # a binary
  type -a cd      # builtin

Other options::

 # +x bit and ./
 chmod +x script.sh
 ./script.sh   # that works if script.sh has #!/bin/bash as a first line
 # with no x bit
 bash script.sh  # this will work even without #!/bin/bash

**Extension is optional** note that *.sh* extension is optional, script may have any name


Functions as part of your environment
-------------------------------------
Alias is a shortcut to a long command, while function is a piece of programming
that has logic and can accept input parameters. Functions can be defined on-the-fly
from the cli, or can go to a file. Let us set *~/bin/functions* and collect
everything useful there.::

 # cd to the directory and lists it at once
 # can be run as: lcd <path/to/directory>
 lcd() {
   cd $1
   ls -FlA
 }
 
 # in one line, note spaces and ; delimiters
 lcd() { cd $1; ls -FlA; }
 # -or- in a full format
 function lcd { cd $1; ls -FlA; }
 
By now function has been defined, to run it, one has to invoke it.::

 source ~/bin/functions
 lcd dir1

The function refers to passed arguments by their position (not by name),
that is $1, $2, and so forth::

 func_name arg1 arg2 arg3  # will become $1 $2 $3

Functions in BASH have ``return`` but it only returns the exit code. Useful
in cases where you want to 'exit' the function and continue to the rest of the script.
By default functions' variables are in the global space, once chaged in the function is
seen everywhere else. ``local`` can be used to localize the vars. Compare::

 var=2; f() { var=3; }; f; echo $var
 var=2; f() { local var=3; }; f; echo $var

If you happened to build a function in an alias way, redefining a command name while
using that original command inside the function, you need to type *command* before
the name of the command, like::

 rm() { command rm -i "$@"; } 

here you avoid internal loops (forkbombs).

Exporting a function with ``export -f function_name`` lets you pass a function to a sub-shell,
by storing that function in a environment variable. Helpful when you want to use it within a 
command substitution, or any other case that launches a subshell, like
``find ... -exec bash -c 'function_name {}' \;``. 


Variables
---------
In shell, variables define your environment. Common practice is that environmental vars are written IN CAPITAL: $HOME, $SHELL, $PATH, $PS1, $RANDOM. To list all defined variables ``printenv``. All variables can be used or even redefined. No error if you call an undefined var, it is just considered to be empty::

 # assign a variable, note, no need for ; delimiter
 var1=100 var2='some string'
 
 # calling a variable is just putting a $ dollar sign in a front
 echo "var1 is $var1"
  
 # re-assign to another var
 var3=$var1
 
 # when appending a variable, it is considered to be a string 
 var+=<string>/<integer>
   var1+=50  # var1 is now 10050
   var2+=' more' # var2 is 'some string more'
 # we come later to how to deal with the integers (Arithmetic Expanssions $(()) below)
 
There is no need to declare things in advance: there is flexible
typing.  In fact, you can access any variable, defined or not.
However, you can still declare things to be of a certain type if you
need to::

 declare -r var=xyz   # read-only
 declare -i var  # must be treated as an integer, 'man bash' for other declare options

BASH is smart enough to distinguish a variable inline without special quoting::

 dir=$HOME/dir1 fname=file fext=xyz echo "$dir/$fname.$fext"

though if variable followed by a number or a letter, you have to
explicitly separate it with the braces syntax::

 echo ${dir}2/${file}abc.$fext

Built-in vars:

 - $?  exit status of the last command
 - $$  current shell pid
 - $#  number of input parameters
 - $0  running script name, full path
 - $FUNCNAME  function name being executed, [ note: actually an array ${FUNCNAME[*]} ]
 - $1, $2 ... input parameter one by one (function/script)
 - "$@" all input parameters as is in one line

::

 f() { echo -e " number of input params: $#\n input params: $@\n shell process id: $$\n script name: $0\n function name: $FUNCNAME"; return 1; }; f arg1 arg2; echo "exit code: $?"

What if you assing a variable to a variable like::

 var2='something'
 var1=\$var2
 echo $var1     # will return '$var2' literally
 
 # BASH provides built-in 'eval' command that reads the string then re-evaluate it
 # if variables etc found, they are given another chance to show themselves
 
 eval echo $var1  # returns 'something'

In more realistic examples it is often used to compose a command string based on input
parameters or some conditionals and then evaluate it at very end.


Magic of BASH variables
-----------------------
BASH provides wide abilities to work with the vars "on-the-fly" with
``${var...}`` like constructions.  This lets you do simple text
processing easily.  These are nice, but are easy to forget so you will
need to look them up when you need them.

 - Assign a $var with default *value* if not defined: ``${var:=value}``
 - Returns $var value or a default *value* if not defined: ``${var:-value}``
 - Print an *error_message* if var empty: ``${var:?error_message}``
 - Extract a substring: ``${var:offset:length}``, example ``var=abcde; echo ${var:1:3}`` returns 'bcd'
 - Variable's length: ``${#var}``
 - Replace beginning part: ``${var#prefix}``
 - Replace trailing part: ``${var%suffix}``
 - Replace *pattern* with the *string*: ``${var/pattern/string}``
 - Modify the case of alphabetic characters: ``${var,,}`` for lower case or ``${var^^}`` for upper case

::

 # will print default_value, which can be a variable
 var=''; echo ${var:-default_value}
 var1=another_value; var='';  echo ${var:-$var1}
 
 # assign the var if it is not defined
 # note that we use ':' no operation command, to avoid BASH's 'command not found' errors 
 : ${var:=default_value}
 
 # will print 'not defined' in both cases
 var='';  echo ${var:?not defined}
 var=''; err='not defined'; echo ${var:?$err}
 
 # will return 'love you'
 var='I love you'; echo ${var:2:8}
 
 # will return 15, that is a number of characters
 var='I love you too!'; echo ${#var}
 
 # returns file.ext
 var=26_file.ext; echo ${var#[0-9][0-9]_}
 
 # in both cases returns 26_file
 var=26_file.ext; echo ${var%.ext}
 var=26_file.ext; echo ${var%.[a-z][a-z][a-z]}
 
 # returns 'I hate you'
 var='I love you'; echo ${var/love/hate}
 # other options for substitutions
 var=' some text ';
 echo ${var/# /}  # returns without the first space
 echo ${var/% /}  # without the last space
 echo ${var// /}  # without spaces at all
 
Except for the *:=* the variable remains unchanged. If you want to
redefine a variable::

  var='I love you'; var=${var/love/hate}; echo $var  # returns 'I hate you'

BASH allows indirect referencing, consider::

 var1='Hello' var2=var1
 echo $var2  # returns text 'var1'
 echo ${!var2}  # returns 'Hello' instead of 'var1'

To address special characters::

 # replacing all tabs with the spaces in the var
 var=${var//$'\t'/ }


[Lecturer's note: ~20 minutes for the hands-on exercises. Solution examples can be given at very end.]

:Exercise 2.2:
 - Expand ``lcd()`` function to have WRKDIR as a default directory in case function is invoked
   without any input parameter.
 - Expand the Exersice 2.1's ``ls ... && echo .. || echo`` example, make a function that check any
   file/directory existense given as an argument, like ``checkexist path/to/file``. If no argument
   given, function must return an error message "File or directory not found".
 - Implement a ``spaceusage()`` function with ``du ... | sort ...`` (see Aliases part examples)
   that takes directory path as an argument, and if missing uses current directory.
 - Using ``find`` utility, implement a 'fast find' function ``ff word``. The function should return a long listing
   (ls -ldA) of any file or directory names that contain the <word>. Make search case insensitive.
   Note: your newly ceated functions should go to *~/bin/functions* file.
 - Write two functions ``get_filename()`` and ``get_extension()``. Both should accept a full filename
   like *path/to/filename.ext* of any length and return *filename* or *ext* correspondingly.
   Extenssion can be of any length. Function should handle missing argument case correctly.
 - Expand *get_filename()* so that it would accept extenssion pattern as a second argument (if given) and
   return *filename* out of *path/to/filename.tar.gz* or alike. I.e. ``get_filename path/to/filename.tar.gz tar.gz``
 - (*) By now one should be able to explain: ``:() { :|:&; };:``. *&* in this case sends process
   to background. [WARNING: it is a forkbomb]
 - (*) On Triton write a function that ``lfs find`` all the dirs/files at $WRKDIR that do not
   belong to your group and fix the group ownership. Use ``find ... | xargs``. Tip: on Triton at
   WRKDIR your username $USER and group name are the same. On any other filesystem, ``$(id -gn)``
   returns your group name. One can 
 - (*) Expand the function above to set group's s-bit on all the $WRKDIR directories.