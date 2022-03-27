Loops
=====

Arithmetic
----------
BASH works with integers only (no floating point) but supports wide range of arithmetic operators using
arithmetic expansion ``$(( expression ))``.

 * All tokens in the expression undergo parameter and variable expansion, command substitution,
   and quote removal. The result is treated as the arithmetic expression to be evaluated.
 * Arithmetic expansion may be nested.
 * Variables inside double parentheses can be without a $ sign.
 * BASH has other options to work with the integers, like ``let``, ``expr``, ``$[]``, and in
   older scripts/examples you may see them.

Available operators:

 - ``n++``, ``n--``, ``++n``, ``--n`` increments/decrements
 - ``+``, ``-`` plus minus
 - ``**`` exponent
 - ``*``, ``/``, ``%`` multiplication, (truncating) division, remainder
 - ``&&``, ``||`` logical AND, OR
 - ``expr?expr:expr`` conditional operator (ternary)
 - ``==``, ``!=``, ``<``, ``>``, ``>=``, ``<=`` comparison
 - ``=``, ``+=``, ``-=``, ``*=``, ``/=``, ``%=`` assignment
 - ``()``  sub-expressions in parentheses  are  evaluated first
 - The full list includes bitwise operators, see ``man bash`` section *ARITHMETIC EVALUATION*.

::

 # without dollar sing value is not returned, though 'n' has been incremented
 n=10; ((n++))

 # but if we need a value
 n=10; m=3; q=$((n**m))

 # here we need exit code only
 if ((q%2)); then echo odd; fi
 if ((n>=m)); then ...; fi

 # condition ? integer_value_if_true : integer_value_if_false
 n=2; m=3; echo $((n<m?10:100))
 
 # checking number of input parameters, if $# is zero, then exit
 # (though the alternative [[ $# == 0 ]] is more often used, and intuitively more clear)
 if ! (($#)); then echo Usage: $0 argument; exit1; fi

::

 # sum all numbers from 1..n, where n is a positive integer
 # Gauss method, summing pairs
 if (($#==1)); then
   n=$1
 else
   read -p 'Give me a positive integer ' n
 fi
 echo Sum from 1..$n is $((n*(n+1)/2))

Left for the exercise: make a summation directly 1+2+3+...+n and compare performance with the above one.

For anything more mathematical than summing integers, one should use something else,
one of the option is ``bc``, often installed by default.

::
  
  # bc -- an arbitrary precision calculator language
  # compute Pi number
  echo "scale=10; 4*a(1)" | bc -l

A way to get percentage with the floating point

::

 done=5; total=12; printf "%.2f%%\n" "$((10**3*100*done/total))e-3"


For loops
---------
BASH offers several options for iterating over the lists of elements. The options include

 * Basic construction ``for arg in item1 item2 item3 ...``
 * C-style *for loop* ``for ((i=1; i <= LIMIT ; i++))``
 * while and until constructs

Simple loop over a list of items:
 
::

 # note that if you put 'list' in quotes it will be considered as one item
 for dir in dir1 dir2 dir3/subdir1; do
   echo "Archiving $dir ..."
   tar -caf ${dir//\/.}.tar.gz $dir && rm -rf $dir
 done

If path expansions used (\*, ?, [], etc), loop automatically lists current directory:

::

 # example below uses ImageMagick's utlity to convert all *.jpg files
 # in the current directory to *.png.
 # i.e. '*.jpg' similar to 'ls *.jpg'
 for f in *.jpg; do
  convert "$f" "${f/.jpg/.png}"   # quotes to avoid issues with the spaces in the name
 done
 
 # another command line example renames *.JPG and *.JPEG files to *.jpg
 # note: in reality one must check that a newly created *.jpg file does not exist
 for f in *.JPG *JPEG; do mv -i "$f" "${f/.*/.jpg}"; done

 # do ... done in certain contexts, can be omitted by framing the command block within curly brackets
 # and certain for loop can be written in one line as well
 for i in {1..10}; { echo i is $i; }

If *in list* omitted, *for* loop goes through script/function input parameters ``$@``

::

 # here is a loop to rename files which names are given as input parameters
 # touch file{1..3}; ./newname file1 file2 file3
 for old; do
   read -p "old name $old, new name: " new
   mv -i "$old" "$new"
 done

Note: as side note, while working with the files/directories, you will find lots of examples where
loops can be emulated by ``find ... -print0 | xargs -0 ...`` pipe.

Loop output can be piped or redirected:

::

 # loop other all users (on a server) to find out who has logged in within last month
 for u in $(getent group triton-users | cut -d: -f4 | tr ',' ' '); do
   echo $u: $(last -Rw -n 1 $u | head -1)
 done | sort > filename

The *list* can be anything what produces a list, like Brace expansion *{1..10}*, command substitution etc.::

 # on Triton, do something to all pending jobs based on squeue output
 for jobid in $(squeue -h -u $USER -t PD -o %A); do
   scontrol update JobId=$jobid StartTime=now+5days
 done

 # using find to make a list of files to deal with; the benefit here is that you work
 # with the filename as a variable, which gives you flexibility as comparing to 
 # 'find ... -exec {}' or 'find ... print0 | xargs -0 ...'
 for f in $(find . -type f -name '*.sh'); do
   if ! bash -n $f &>/dev/null; then
     mv $f ${f/.sh/.fixme.sh}
   fi
 done

C-style, expressions evaluated according to the arithmetic evaluation rules::

 N=10
 for ((i=1; i <= N ; i++))  # LIMIT with no $
 do
   echo -n "$i "
 done

Loops can be nested.

While/until loops
-----------------

Other useful loop statement are ``while`` and ``until``. Both execute continuously as long as the
condition returns exit status zero/non-zero correspondingly.

::

 while condition; do
   command1
   command2
   ...
 done

 # sum of all numbers 1..n; n expected as an argument
 n=$1 i=1
 until ((i > n)); do
   ((s+=i)); ((i++))
 done
 echo Sum of 1..$n is $s

 # endless loop, can be stopped with Ctrl-c or killed
 # drop an email every 10 minutes about running jobs on Triton
 # can be used in combination with 'screen', and run in background
 while true; do
   squeue -t R -u $USER | mail -s 'running jobs' mister.x@aalto.fi
   sleep 600
 done

 # with the help of 'read var' passes file line by line,
 # IFS= variable before read command to prevent leading/trailing
 # whitespace from being trimmed
 input='/path/to/txt/file'
 while IFS= read -r line; do
  echo $line
 done < "$input"

 # reading file fieldwise, IFS= is a delimiter, note quoting with \
 file='/path/to/file.csv'
 while IFS=\; read -r f1 f2 f3 f4; do
   printf 'Field1: %s, Field2: %s, Field4: %s\n' "$f1" "$f2" "$f4"
 done < "$file"

 # process substitution
 while IFS= read -r line; do
   # do something with the lines
 done < <(file -b *)
 # instead, one can mistakenly try 'file -b * | while read line; do ... done'
 # with pipe, 'while' body will be run in a subshell, and thus all variables
 # used inside the loop will die when loop is over

All the things mentioned above for *for* loop applicable to ``while`` / ``until`` loops.

*printf* should be familiar to programmers, allows formatted output
 similar to C printf. [#printf]_

Loop control
------------

Normally *for* loop iterates until it has processed all its input arguments.
*while* and *until* loops iterate until the loop control returns a certain status. But if
needed, one can terminate loop or jump to a next iteration.

 - ``break`` terminates the loop
 - ``continue`` jump to a new iteration
 - ``break n`` will terminate *n* levels of loops if they are nested, otherwise terminated only
     loop in which it is embedded. Same kind of behaviour for ``continue n``.

Even though in most of the cases you can design the code to use conditionals or alike,
*break* and *continue* certainly add the flexibility.

::

 # here we expand an earlier example to avoid errors in case $f is missing/not accesible
 for f in *.JPG *.JPEG; do
   [[ -r "$f" ]] || { echo "$f is missing on inaccessible"; continue; }
   mv -i "$f" "${f/.*/.jpg}"
 done

Exercise 2.4
------------

.. exercise::

 - Using ``for d; do ... done`` expand *tarit.sh* so that it would accept multiple directories. 
   If no directory given, it still suppose to archive the current one.
 - Using ``for`` loop rename all the files in the current directory tree with the *.txt* extension to *.fixed.txt*.
   Step #1: create dummy .txt files first with ``mkdir d{1..3}; touch d{1..3}/{1..3}.txt``.
   Step #2: combine 'for' loop with 'find': ``for f in $(find . -name '*.txt'); do ... done``.
 - Use this page ``while`` example with *.cvs*, take the *demospace/Finnish_Univ_students_2018.csv*
   to count total number of students around Finland. Tip: add checking that the number of
   students field is a number ``[[ $totalnmb =~ ^[0-9]+$ ]]``
 - Using built-in arithmetic write a script *daystill.sh* that counts a number of days till a
   deadline (vacation/salary). 
   Script should take a date as an argument, where date format is ``days_till 2019-6-1``.
   Tip #1: ``date -d GIVEN_DATE +"%s"`` returns number of seconds till GIVEN_DATE since
   1970-01-01 00:00:00 UTC. ``date +%s`` returns seconds till now.
   Tip #2: enough if you convert seconds to a number of days roughly ``$(( ... /60/60/24))``.
 - Make script that takes a list of files and checks if there are files in there with
   the spaces in the name, and if there are, rename them by replacing spaces with the underscores.
   Use BASH's builtin functionality only.
   
   - As a study case, compare it against
     ``find . -depth -name '* *' -execdir rename 's/ /_/g' {} \;``
   
 - Write separate scripts that count a sum of any *1+2+3+4+..+n*
   sequence, both the Gauss version (see above) and summation with ``for ((...))``.  Where
   *n* is an argument, like *gauss.sh 1000*. Benchmark them with ``time`` for n=10000 or
   more.

   - (*) Implement both methods within one script as functions and benchmark them whithin the file
   - (*) For the direct summation one can avoid loops, how? Tip: discover ``eval $(echo {1..$n})``

 - (*) Get familiar with the ``getent`` and ``cut`` utilities. Join them with a loop construction 
   to write a *mygetentgroup* script or just a oneliner
   that generates a list of users and their real names that belong to a given group. Like::
   
     $ mygetentgroup group_name
     meikalaj1: Jussi Meikäläinen
     meikalam1: Maija Meikäläinen
     ...``
 - (*) To Aalto users: on kosh/lyta run ``net ads search samaccountname=$USER accountExpires 2>/dev/null``
   to get your account expiration date. It is a 18-digit timestamp, the number of 100-nanoseconds
   intervals since Jan 1, 1601 UTC. Implement a function that accept a user name, and if not given
   uses current user by default, and then converts it to the human readable time format.
   Tip: http://meinit.nl/convert-active-directory-lastlogon-time-to-unix-readable-time

   - Expand it to handle "Got 0 replies" response, i.e. account name not found.


.. [#printf] https://wiki.bash-hackers.org/commands/builtin/printf
