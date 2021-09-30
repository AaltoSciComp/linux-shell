Linux shell tutorial
====================

Linux Shell tutorial by Science IT at Aalto University.

This course consists of two parts: Linux Shell Basics and Linux Shell Scripting. The first one
covers introductory level shell usage (which also is a backdoor introduction to Linux basics). 
The second one covers actual BASH scripting, using learning by doing.

Starred exercises (*) are for advanced users who would like further stimulation.

.. prereq::

   - Part #1: A Linux/Mac computer or a Windows with SSH client installed to access any Linux server
   - Part #2: Shell basics, know how to create a directory and edit a file from command line


Who is the course for?
----------------------

- Part #1 for those who need a comprehensive dive into shell basics.
- Part #2 for people with intermediate/advanced level of Linux/Mac shell


Credits
-------

1 credit if completed together with Linux shell basics -course and all 
exercises completed and sent to scip@aalto.fi.


Schedule
--------

Ongoing. Optional lectures on average once per year with following timetable:

.. csv-table::
   :widths: auto
   :delim: ;

   Linux Shell Basics ; 2 sessions x 2h
   Linux Shell Scripting ; 4 sessions x 3h, session rough schedule 2x1h25m with 10m break in between.


Setting up instructions for the lecturer
----------------------------------------

Main terminal white&black with the enlarged font size. One small terminal at the top that shows
commands to the learners.

 - ``export PROMPT_COMMAND='history -a'``   # .bashrc or all the terminals one launches commands
 - ``tail -n 0 -F .bash_history``

Alternatively, ``script`` allows to follow the session even after sshing to a remote host
plus command appear as soon as they are run. The
regular expression can be adapted to the lecturer's PS1, this one assumes *]$ command*.

 - ``script -f demos.out``   # action window
 - ``tail -n 1 -f demos.out | while read line; do [[ "$line" =~ \]\$\ ([^ ].+)$ ]] && echo ${BASH_REMATCH[1]}; done``


References
----------

Based on
 - ``man bash`` v4.2 (Triton default version in Feb 2018)
 - Advanced BASH scripting guide [#absguide]_
 - UNIX Power Tools, Shelley Powers etc, 3rd ed, O'Reilly
 - common sense and 20+ years Linux experience
 - see also other references in the text

   .. [#absguide] http://tldp.org/LDP/abs/html/index.html
   .. [#putty] https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
   .. [#xming] http://www.straightrunning.com/XmingNotes/
   .. [#ps1] https://www.ibm.com/developerworks/linux/library/l-tip-prompt/
   .. [#find1] https://alvinalexander.com/unix/edu/examples/find.shtml
   .. [#find2] http://www.softpanorama.org/Tools/Find/index.shtml
   .. [#putty-sshkeys] https://the.earth.li/~sgtatham/putty/0.70/htmldoc/
   .. [#umask] https://www.computerhope.com/unix/uumask.htm
   .. [#printf] https://wiki.bash-hackers.org/commands/builtin/printf
   .. [#profiling] https://stackoverflow.com/questions/5014823/how-to-profile-a-bash-shell-script-slow-startup



.. toctree::
   :caption: PART #1. Linux shell basics
   :hidden:

   the_shell
   starting_commands
   processes
   files_and_directories
   find
   file_archive_transfer
   hotkeys
   files_continue
   input_output
   pipelines_grep

.. toctree::
   :caption: PART #2. Linux shell scripting
   :hidden:

   quoting_substitution_aliases
   variables_functions_environments
   conditionals
   loops
   arrays_inputs_here_documents
   traps_debugging_profiling
   parallel_crontab_perlawksed
   references


.. toctree::
   :caption: About
   :hidden:

   contact
   about

.. toctree::
   :caption: Other
   :hidden:

   to_continue
   bonus_material