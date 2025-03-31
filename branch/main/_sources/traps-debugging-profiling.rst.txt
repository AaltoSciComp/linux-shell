Traps, debugging, profiling
===========================

Catching kill signals: trap
---------------------------
What if your script generates temp file and you'd like to keep it clean
even if script gets interrupted at the execution time?

The built-in ``trap`` command lets you tell the shell what to do if your script received
signal to exit. It can catch all, but here listed most common by their
numbers.  Note that signals are one of the common ways of communicating
with running processes in UNIX: you see these same numbers and names
in programs like ``kill``.

 * 0  EXIT  exit command
 * 1  HUP   when session disconnected
 * 2  INT   interrupt - often Ctrl-c
 * 3  QUIT  quit - often Ctrl-\
 * 9  KILL  real kill command, it can't be caught
 * 15 TERM  termination, by ``kill`` command

::

 # 'trap' catches listed signals only, others it silently ignores
 # Usage: trap group_of_commands/function list_of_signals

 trap 'echo Do something on exit' EXIT
 
Expanding the backup script from the Arrays section, this can be added to the very
beginning:: 
 
 interrupted() {
   echo 'Seems that backup has been interrupted in the middle'
   echo 'Rerun the script later to let rsync to finish its job'
   exit 1
 }
 
 trap interrupted 1 2 15
 # ... the rest of the script
 
In other situation, instead of *echo*, one can come up with something else:
removing temp files, put something to the log file or output a
valuable error message to the screen.

**Hint** About signals see *Standard signals* section at ``man 7 signal``. Like Ctrl-c is INT (aka SIGINT).


Debugging and profiling
-----------------------
BASH has no a debugger, but there are several ways to help with the debugging

Check for syntax errors without actual running it ``bash -n script.sh``

Or echos each command and its results with ``bash -xv script.sh``, or even adding options directly
to the script. ``-x`` enables tracing during the execution, ``-v`` makes bash to be verbose. Both
can be set directly from the command line as above or with ``set -xv`` inside the script.

::

 #!/bin/bash -xv

To enable debugging for some parts of the code only::

  set +x
  ... some code
  set -x

If you want to check quickly a few commands, with respect to how variables or other
substitutions look like, use DEBUG variable set to *echo*.

::

 #!/bin/bash

 $DEBUG command1 $arguments
 command2

 # call this script like 'DEBUG=echo ./script.sh' to see how *command1* looks like
 # otherwise the script can be run as is.


One can also ``trap`` at the EXIT, this should be the very first lines in the script::
 
 end() { echo Variable Listing: a = $a  b = $b; }
 trap end EXIT  # will execute end() function on exit
 
For a sake of profiling one can use PS4 and ``date`` (GNU version that deals with nanoseconds). PS4 is
a built in BASH variable which is printed before each command bash displays during an execution trace.
The first character of PS4 is replicated multiple times, as necessary, to indicate multiple levels
of indirection. The default is ``+``. Add the lines below right after '#!/bin/bash'

::

 # this will give you execution time of each command and its line number
 # \011 is a tab
 PS4='+\011$(date "+%s.%N")\011${LINENO}\011'
 set -x
 
Optionally, if you want tracing output to be in a separate file::

 PS4='+\011$(date "+%s.%N")\011${LINENO}\011'
 exec 5> ${0##*/}.$$.x && BASH_XTRACEFD='5' && set -x

Or to get your script looking more professional, one can enable DEBUG, i.e. tracing only
happens when you run as ``DEBUG=profile ./script.sh``::

 case $DEBUG in
   profile|PROFILE|p|P)
     PS4='+\011$(date "+%s.%N")\011${LINENO}\011'
     exec 5> ${0##*/}.$$.x && BASH_XTRACEFD='5' && set -x ;;
 esac

For the larger scripts with loops and functions tracing output with the date stamps and line numbers
can be summarized. For further discussion please take a look at [#profiling]_





.. [#profiling] https://stackoverflow.com/questions/5014823/how-to-profile-a-bash-shell-script-slow-startup
