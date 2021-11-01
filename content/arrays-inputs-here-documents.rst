Arrays, input, Here Documents
=============================

Arrays
------
BASH supports both indexed and associative one-dimensional arrays. Indexed array can be declared
with ``declare -a array_name``, or first assignment does it automatically (note: indexed arrays only)::

 arr=(my very first array)
 arr=('Otakaari 1' Espoo 02150 [6]='PL 11000')
 arr[5]=AALTO

To access array elements (the curly braces are required, unlike normal
variable expansion)::

 # elements one by one
 echo ${arr[0]} ${arr[1]}
 
 # array values at once
 ${arr[@]} 
 
 # indexes at once
 ${!arr[@]}
 
 # number of elements in the array
 ${#arr[@]}
 
 # length of the element number 2
 ${#arr[2]}

 # to append elements to the end of the array
 arr+=(value)

 # assign a command output to array
 arr=($(command))
 
 # emptying array
 arr=()
 
 # to destroy, delete an array
 unset arr

 # to unset a single array element
 unset arr[6]

 # sorting array
 IFS=$'\n' sorted=($(sort <<<"${arr[*]}"))
 
 # array element inside arithmetic expanssion requires no ${}
 ((arr[$i]++))
 
 # split a string like 'one two three etc' or 'one,two,three,etc' to an array
 # note that IFS=', ' means that separator is either space or comma, not a sequence of them
 IFS=', ' read -r -a arr <<< "$string"
 
 # spliting a word to an array letter by letter
 word=qwerty; arr=($(echo $word | grep -o .))

Loops through the indexed array::

 for i in ${!arr[@]}; do
   echo arr[$i] is ${arr[$i]}
 done

Negative index counts back from the end of the array, *[-1]* referencing to the last element.

Quick ways to print array with no loop::

 # with keys, as is
 declare -p arr
 
 # indexes -- values
 echo ${!arr[@]} -- ${arr[@]}

 # array elements values one per line
 printf "%s\n" "${arr[@]}"

Passing an array to a function as an argument could be the use case when you want to make it local::

 f() {
   local arr=(${!1})    # pass $1 argument as a refence
   # do something to array elements
   echo ${arr[@]}
 }
 
 # invoke the function, huom that no changes have been done to the original arr[@]
 arr=(....)
 f arr[@]
 

BASH associative arrays (this type of array supported in BASH since version 4.2) needs to be
declared first (!) ``declare -A asarr``.

Both indexed arrays and associative can be declared as an array of integers, if all elements
values are integers ``declare -ia array`` or ``declare -iA``. This way element values are
treated as integers always.

::

 asarr=([university]='Aalto University' [city]=Espoo ['street address']='Otakaari 1')
 asarr[post_index]=02150

Addressing is similar to indexed arrays::

 for i in "${!asarr[@]}"; do
   echo asarr[$i] is ${asarr[$i]}
 done

Even though key can have spaces in it, quoting can be omitted.

::

 # use case: your command returns list of lines like: 'string1 string2'
 # adding them to an assoative array like: [string1]=string2
 declare -A arr
 for i in $(command); do
   arr+=(["${i/ */}"]="${i/* /}")
 done

Variable expanssions come out in the new light::

 # this will return two elements of the array starting from number 1
 ${arr[@]:1:2}

 # all elements without last one
 ${arr[@]:0:${#arr[@]}-1}
 
 # parts replacement will be applied to all array elements
 declare -A emails=([Vesa]=vesa@aalto.fi [Kimmo]=kimmo@helsinki.fi [Anna]=anna@math.tut.fi)
 echo ${emails[@]/@*/@gmail.com}
 # returns: vesa@gmail.com anna@gmail.com kimmo@gmail.com
 
For a sake of demo: let us count unique users and their occurances (yes, one can do it with 'uniq -c' :)

::

 # declare assoative array of integers
 declare -iA arr

 for i in $(w -h | cut -c1-8); do   # get list of currenly logged users into loop
   for u in ${!arr[@]}; do   # check that they are unique
     if [[ $i == $u ]]; then
       ((arr[$i]++))
       continue 2 
     fi 
   done
   arr[$i]=1  # if new, add a new array element
 done

 for j in ${!arr[@]}; do    # printing out
   echo ${arr[$j]} $j
 done
 
Another working demo: script that automates backups or just makes a sync of data to a remote server.
Same can be adapted to copy locally, to a usb drive or alike.

::

 # array of directories to be backuped, to skip one, just comment with #
 declare -A dirs
 dirs[wlocal]=/l/$USER
 dirs[xpproject]=/m/phys/extra/project/xp
 dirs[homebin]=$HOME/bin
  
 cmd='/usr/bin/rsync'                   # rsync 
 args="-auvW --delete --progress $@"    # accept extra args, like '-n' for the dryrun first
 serv='user@server:backups'             # copying to ~/backups that must exist
 
 # array key is used for the remote dir name
 for d in ${!dirs[@]}; do
   echo "Syncing ${dirs[$d]}..."
   $cmd $args ${dirs[$d]}/ $serv/$d
 done


Exercise 2.5
------------

.. exercise::

 - make a script/function that produces an array of random numbers, make sure that numbers
   are unique. Print the array nicely using ``printf`` for formating.
 
   - one version should use BASH functionality only (Tip: ``$RANDOM``)
   - the other one can use ``shuf``

 - (*) Pick up the ``ipvalid`` function that we have developed earlier, implement IP matching
   regular expression as ``^([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})$`` and
   work with the ${BASH_REMATCH[*]} array to make sure that all numbers are in the range 0-255


Working with the input
----------------------
User input can be given to a script in three ways:

 * as command arguments, like ``./script.sh arg1 arg2 ...``
 * interactively from keyboard with ``read`` command
 * as standard input, like ``command | ./script``

Nothing stops from using a combination of them or all of the approaches in one script.
Let us go through the last two first and then get back to command line arguments.

``read`` can do both: read from keyboard or from STDIN

::

 # the command prints the prompt, waits for the response, and then assigns it
 # to variable(s)
 read -p 'Your names: ' firstn lastn
 
 # read into array, each word as a new array element ('arr' declared automatically)
 read -a arr -p 'Your names: '
 
Given input must be checked (!) with a pattern, especially if script creates directories,
removes files, sends emails based on the input.

::

 # request a new directory name till correct one is given (interrupt with Ctrl-C)
 regexp='^[a-zA-Z0-9/_-]+$'
 until [[ "$newdir" =~ $regexp ]]; do
   read -p 'New directory: ' newdir
 done

 
``read`` selected options

 * ``-a <ARRAY>``  read the data word-wise into the specified array <ARRAY> instead of normal variables
 * ``-N <NCHARS>`` reads <NCHARS> characters of input, ignoring any delimiter, then quits
 * ``-p <PROMPT>`` the prompt string <PROMPT> is output (without a trailing automatic newline) before the read is performed
 * ``-r``  raw input - disables interpretion of backslash escapes and line-continuation in the read data
 * ``-s`` secure input - don't echo input if on a terminal (passwords!)
 * ``-t <TIMEOUT>`` wait for data <TIMEOUT> seconds, then quit (exit code 1)

``read`` is capable of reading STDIN, case like ``command | ./script``, with ``while read var`` it goes
through the input line by line::

 # IFS= is empty and echo argument in quotes to make sure we keep the format
 # otherwise all spaces and new lines shrinked to one and leading/trailing whitespace trimmed
 while IFS= read -r line; do
   echo "line is $line"    # do something useful with $line
 done

 # To check current $IFS
 cat -A <<<"$IFS"

Though in general, whatever comes from STDIN can be proceeded as::

 # to check that STDIN is not empty
 if [[ -p /dev/stdin ]]; then
   # passing STDIN to a pipeline  (/dev/stdin can be omitted)
   cat /dev/stdin | cut -d' ' -f 2,3 | sort
 fi

Other STDIN tricks that one can use in the scripts::

 # to read STDIN to a variable, both commands do the same
 var=$(</dev/stdin)
 var=$(cat)
 
In the simplest cases like ``./script arg1 arg2 ...``, you check *$#* and then assign
*$1, $2, ...* the way your script requires.

::

 # here we require exactly two arguments
 if (($#==2)); then
   var1=$1 var2=$2
   # ... do something useful
 else
   echo 'Wrong amount of arguments'
   echo "Usage: ${0##*/} arg1 arg2"
   exit 1
 fi
 
To work with all input arguments at once we have *$@*::

 # $# is a number of arguments on the command line, must be non-zero
 if (($#)); then
   for i; do
     echo "$i"
     # ... do something useful with each element of $@
     # note that 'for ...' uses $@ by default if no other list given with 'in ...'
   done
 else
   echo 'No command line arguments given'
 fi

As a use case, our *tarit.sh* script. The script can accept STDIN and
arguments, so we check both::

 # Usage: tarit.sh [dirname1 [dirname2 [dirname3 ...]]]
 # or     command | tarit.sh
 
 # by default no directories to archive. i.e. current
 args=''
 
 # checking for STDIN, if any, assigning STDIN to $args
 [[ -p /dev/stdin ]] && args=$(</dev/stdin)
 
 # if arguments are given, appending the $args with $@
 (($#)) && args+=" $@"
 
 # no arguments, no stdin, then it is a current dir
 [[ -z "$args" ]] && args="$(pwd)"
 
 # by now we should have a directory list in $args to archive
 for d in $args; do
   # checking that directory exists, if so, archive it
   if [[ -d "$d" ]]; then
     echo Archiving $d ...
     tar caf ${d##*/}.$(date +%Y-%m-%d).tar.gz "$d"
   else
     echo "   $d does not exist, skipping."
   fi
 done

Often, the above mentioned ways are more than enough for simple scripts.
But what if options and arguments are like
``./script [-f filename] [-z] [-b] [arg1 [arg2 [...]]]`` or more complex?
(common notaion: options in the square brackets are optional). What if you write
a production ready script that will be used by many other as well?

It is were ``getopt`` offers a more efficient way of handling script's input options.
In the simplest case ``getopt`` command (do not get confused with ``getopts`` built-in BASH
function of similar kind) requires two parameters to work:
first is a list of valid input options -- sequence of letters and colons. If letter
followed by a colon, the option requires an argument, if folowed by two colons, argument
is optional. For example, the string ``getopt "sdf:"`` says that the options -s, -d and -f
are valid and -f requires an argument, like *-f filename*.
The second argument required by  *getopt* is a list of input parameters (options + arguments)
to check, i.e. just ``$@``.

Let us use *cx* script as a demo:

::

 # common usage function with the exit at the end
 usage() {
   echo "Usage: $sname [options] file [file [file...]]"
   echo '       -a, gives access to all, like a+x, by default +x'
   echo '       -d <directory/path/bin>, path to the bin directory'
   echo "          can be used in 'cx' to copy a new script there"
   echo '       -a, gives access to all, like a+x, by default +x'
   echo '       -d <directory/path/bin>, path to the bin directory'
   echo "          can be used in 'cx' to copy a new script there"
   echo '       -v, verbose mode for chmod'
   echo '       -h, this help message'
   exit 1
 }
 
 # whole trick is in this part: getopt validates the input parameters,
 # structures them by dividing options and arguments with --,
 # and returns them to a variable
 # then they are reassigned back to $@ with 'set --'
 opts=$(getopt "avhd:" "$@") || usage
 set -- $opts

 # defining variables' default values
 ALL=''
 CMD='/usr/bin/chmod'
 sname=${0##*/}  # the name this script was called by

 # by now we have a well structured $@ which we can trust.
 # to go through options one by one we start an endless 'while' loop
 # with the nested 'case'. 'shift' makes another trick, every time
 # it is invoked it is equal to 'unset $1', thus $@ arguments are
 # "shifted down", $2 becomes $1, $3 becomes $2, etc
 # 'getopt' adds -- to $@ which separates valid options and the rest
 # that did not qualify, when it comes to '--' we 'break' the loop
 while true; do
   case ${1} in
     -h) usage ;; # output help message and exit
     -a) ALL=a ;; # if -a is given we set ALL
     -v) CMD+=' -v' ;; # if verbose mode required
     -d) shift # shift to take next item as a directory path for -d
         BINDIR="$1"
         if [[ -z "$BINDIR" || ! -d "$BINDIR" ]]; then
           echo "ERROR: the directory does not exist"
           usage
         fi
      ;;
     --) shift; break ;;   # remove --
   esac
   shift
 done

 # script body
 
 case "$sname" in
   cx*) $CMD ${ALL}+rx "$@" && \
        [[ -n "$BINDIR" ]] && cp -p $@ $BINDIR ;;
   cw*) $CMD ${ALL}+w "$@" ;;
   cr*) $CMD ${ALL}+r "$@" ;;
   c-w*) $CMD ${ALL}-w "$@" ;;
   *) echo "ERROR: no idea what $sname is supposed to do"; exit 1 ;;
 esac

``getopt`` can do way more, go for ``man getopt`` for details, as an example::

 # here is getopt sets name with '-n' used while reporting errors: our script name
 # accepts long options like '--filename myfile' along with '-f myfile'
 getopt -n $(basename $0)  -o "hac::f:" --long "help,filename:,compress::"  -- "$@"



Exercise 2.6
------------

.. exercise::

 - Using the latest *tarit.sh* (see lecture notes) version as an example,
   expand above *cx* script to
   accept STDIN, like ``command | cx [options]``, where ``command`` produces a list of
   files. Example ``find . -t file -name '*.sh' | cx -a -d /path/to/bin``.
 - Using *cx* demo as an example, expand the latest version of our *tarit.sh*
   (see lecture notes) to make it accepting the following options and arguments:
   ``tarit.sh -h -y -d <directory/with/backups> [dirname1 [dirname2 [dirname3 ...]]]``.
   By default, with no args, it still should make an archive of the current directory.
   ``-h`` returns usage info, ``-d <directory/path/with/backups>`` is a directory the
   tar archives will go to, your script has to check that directory exists, the
   script must also check whether a newly created archive already exist and if so, skip
   creating the archive with the corresponding warning message.

    - (*) ``-y`` should force overwriting already existing archive.
    - (*) ``-s`` should make script silent, so that no errors or other messages
      would come from any inline command.


Here Document, placeholders
---------------------------

A 'here document' and 'here string' take the line(s) following and send them to standard
input. It's a way to send larger blocks to stdin.

::

 # instead of 'echo $STRING | command ...'
 command <<<$STRING

 # instead of 'cat file | command ...'
 command <<SomeMagicStopWord
 The benefit is that one can use $var, $() etc in the text
 The text ends with the Stop Word on a new line, the word can be any
 SomeMagicStopWord

Often used for messaging, be it an email or dumping bunch of text to file.::

 # NAME, SURNAME, EMAIL, DAYS are set earlier 

 mail -s 'Account expiration' $EMAIL<<END-OF-EMAIL
 Dear $NAME $SURNAME,

 your account is about to expire in $DAYS days.

 $(date)

 Best Regards,
 Aalto ITS
 END-OF-EMAIL

Or just outputting to a file (same can be done with echo commands)::

 cat <<EOF >filename
 ... text
 EOF

One trick that is particularly useful is using this to make a long
comment::

 : <<\COMMENTS
 here come text that is seen nowhere
 there is no need to comment every single line with #
 COMMENTS

**Hint** ``<<\LimtiString`` to turn off substitutions and place text as is with $ marks etc

In case you have a template file which contains variables as placeholders, replacing them::

 # 'template' file like:
 The name is $NAME, the email is $EMAIL
 
 # command to substitute the placeholders and redirect to 'output' file
 # the original 'template' file remains as is
 NAME=Jussi EMAIL=jussi@gmail.com
 cat template | while IFS= read -r line; do eval echo $line; done > output
 # resulting file: The name is Jussi, the email is jussi@gmail.com
