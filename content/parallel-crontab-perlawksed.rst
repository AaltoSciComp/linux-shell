Parallel, crontab, perl/awk/sed
===============================

Running in parallel with BASH
-----------------------------
The shell doesn't do parallelzation in the HPC way (threads, MPI), but
can run some simple commands at the same time without interaction.

The simplest way of parallelization is sending processes to a background and waiting in
the script till they are done.::

 
 # in the script body one may run several processes, like
 command1 &
 command2 &
 command3 &
 
Here is an example that can be run as ``time script`` to demonstrate that execution takes
5 seconds, that is the timing of the longest chunk, and all the processes are run in
parallel and finished before script's exit.:: 

 # trap is optional, just to be on the safe side
 # at the beginning of the script, to get child processes down on exit
 trap 'killall $(jobs -p) 2>/dev/null' EXIT

 # dummy sleep commands groupped with echo and sent to the background
 for i in 1 3 5; do
   { sleep $i; echo sleep for $i s is over; } &
 done
 
 # 'wait' makes sure jobs are done before script is finished
 # try to comment it to see the difference
 wait
 echo THE END


Putting ``wait`` at very end of the script makes it to wait till all the child processes are
over and only then exit. Having ``trap`` at very beginning makes sure we kill all the process
whatever happens to the script. Otherwise they may stay running on their own even if the script has
exited.

Another way to run in parallel yet avoiding sending to the background is using ``parallel``.
This utility runs  the  specified  command, passing it a single one of the specified arguments.
This is repeated for each argument. Jobs may be run in parallel. The default is to run one job per CPU.
If no command is specified before the ``--``, the commands after it are instead run in parallel.

::

 # normally the command is passed the argument at the end of its command line. With -i
 # option, any instances of "{}" in the command are replaced with the argument.
 parallel command {} -- arguments_list   

 # example of making a backup with parallel rsync
 parallel -i rsync -auvW {}/ user@server:{}.backup -- dir1 dir2 dir3

 # in case you want to run a command, say ten times, the arguments can be any dummy list
 # normally parallel passes arguments at the end of the command, with '-i' they needs to 
 # be placed explicitly with '{}', or can be skipped, like here
 parallel -i date -- {1..10}
 
 #  if no command is specified before the --, the commands after it are instead run in parallel,
 parallel -- ls df "echo hi"

On Triton we have installed Tollef Fog Heen's version of parallel from *moreutils-parallel* CentOS' RPM.
GNU project has its own though, with different syntax, but of exactly the same name, so do not get
confused.


Crontab
-------
Allows run tasks automatically in the background. Users may set their own crontabs. Once crontab
task is set, it will run independently on whether you are logged in to the system or not.

Run ``crontab -l`` to list all your current cron jobs, ``crontab -e`` to start editor. When in, you may add
one or several lines, then save what you have added and exit the editor normally.

::

 # run 'script' daily at 23:30
 30 23 * * * $HOME/bin/backup_script > /home/user/log/backup.log 2>&1

 # every two hours on Mon-Wed,Fri
 0 /2 * * 1-3,5 rm /path/to/my/tmp/dir/* >/dev/null 2>&1

The executable script could be a normal command, but crontab's shell has quite limited functionality,
in case of anything more sofisticated than just a single command and a redirection you end up writing
a separate script.

The first five positions corresponds to:

 - minute (0-59)
 - hour (0-23)
 - day (1-31)
 - month (1-12)
 - day of week (0-7, 0 or 7 is Sunday)

Possible values are:

 - ``*`` any value
 - ``,`` value list separator
 - ``-`` range of values
 - ``/`` steps

You set your favorite editor:  ``export EDITOR=vim`` (can be a part of ~/.bashrc).

As part of the crontab file you may set several environmet variables, like
``MAILTO=name.surname@aalto.fi`` to recieve an output from the script or any possible errors.
If *MAILTO* is defined but empty (``MAILTO=""``), no mail is sent.


Perl, awk, sed
--------------
Powerful onliners. Please consult correspoding man pages and other docs for the details,
here we provide some examples. As it was standed at very beginning of the course, shell, with all its
functionality is only a glue in between all kind of utilities, like ``grep``, ``find``, etc.
Perl, awk and sed are what makes terminal even more powerful. Even though Perl can do everything what
can awk and sed, one still may find tons of examples with the later ones. Here we provide some of them.

Python is yet another alternative.

::

 # set delimiter to : and prints the first field of each line of passwd file (user name)
 awk -F: '{print $1}' /etc/passwd
 
 # sort lines by length, several ways to do it
 cat file | perl -e 'print sort { length($a) <=> length($b) } <>'
 cat file | awk '{ print length, $0 }' | sort -n -s | cut -d" " -f2-
 
 # placeholders replacement example above could be 
 NAME=Jussi EMAIL=jussi@gmail.com; sed -e "s/\$NAME/$NAME/" -e "s/\$EMAIL/$EMAIL/" template
 
 # inline word replacing in all files at once
 perl -i -p -e "s/TKK/Aalto/g" *.html


About homework assignments
--------------------------
Available on Triton. See details in the *$course_directory*.