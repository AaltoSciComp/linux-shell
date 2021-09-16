Processes
=========

What's a UNIX process?
----------------------
- To understand a shell, let's first understand what processes are.
- All programs are a process: process is a program in action.
- Processes have:

  - Process ID (integer)
  - Name (command being run)
  - Command line arguments
  - input and output: ``stdin`` (input, like from keyboard),
    ``stdout`` (output, like to screen), ``stderr`` (like stdout)
  - Return code (integer) when complete
  - Working directory
  - environment variables: key-values which get inherited across processes.

- These concepts bind together *all* UNIX programs, even graphical ones.

Process listing commands (feel free to try, but we play more with them later)::

  top              # (q to quit)
  htop             # (q to quit)
  pstree
  pstree $USER
  pstree -pau $USER
  ps auxw

You can find info about your user (try them right away)::

  id
  echo $SHELL

Is your default shell is a ``/bin/bash``? Login to kosh/taltta and run ``chsh -s /bin/bash``

Another way to find out what SHELL you are running::

  ps -p $$

Where am I: ``pwd`` (this shows the first piece of process
information: current directory)

Working with processes
----------------------
All processes are related, a command executed in shell is a child process of
the shell. When child process is terminated it is reported back to parent process.
When you log out all shell child processes terminated along with the
shell.  You can see the whole family tree with ``ps af``.
One can kill a process or make it "nicer".

::

  pgrep -af <name>
  kill <PID>
  pkill <name>
  renice #priority <PID>

Making process "nicer", ``renice 19 <PID>``, means it will run only when nothing
else in the system wants to.
User can increase nice value from 0 (the base priority) up to 19. It is 
useful when you backup your data in background or alike.


Foreground and background processes
-----------------------------------
The shell has a concept of foreground and background processes: a
foreground process is directly connected to your screen and
keyboard. A background process doesn't have input connected.  There
can only be one foreground at a time (obviously).

If you add ``&`` right after the command will send the process to
background. Example: ``firefox --no-remote &``, and same can be done with
any terminal command/function, like ``man pstree &``.  In the big
picture, the ``&`` serves the same role as ``;`` to separate commands,
but backgrounds the first and goes straight to the next.

If you have already running process, you can background with Ctrl-z and then
``bg``. Drawback: there is no easy way to redirect the running task
output, so if it generates output it covers your screen.

List the jobs running in the background with ``jobs``, get a job back
online with  ``fg`` or ``fg <job_number>``. There can be multiple
background jobs.

Kill a foreground job: Ctrl-c

**Hint:** For running X Window apps while you logged in from other
Linux / MacOS make sure you use ``ssh -X ...`` to log in. For Windows users,
you need to install Xming [#xming]_ on your workstation.

**Hint:** For immediate job-state change notifications, use ``set notify``. To automatically
stop background processes if they try writing to the screen ``stty tostop``


Exit the shell and 'screen' utility
-----------------------------------
``logout`` or Ctrl-d (if you don't want Ctrl-d to quit, set ``export IGNOREEOF=1`` in *.bashrc*).

Of course, quitting your shell is annoying, since you have to start
over.  Luckily there are programs so that you don't have to do this.
In order to keep your sessions running while you logged out, you
should learn about the ``screen`` program.

 - ``screen`` to start a session
 - *Ctrl-a d* to detach the session while you are connected
 - ``screen -ls`` to list currently running sessions
 - ``screen -rx <session_id>`` to attach the session, one can use TAB for the autocompletion or skip the <session_id> if there is only one session running
 - ``tmux`` is a newer program with the same style.  It has some extra
   features and some missing features still.

Some people have their ``screen`` open forever, which just keeps
running and never gets closed.  Wherever they are, they ssh in,
connect, and resume right where they left off.

Example: ``irssi`` on kosh / lyta


[Lecture notes: that should be a first half, then joint hands-on/break ~30 mins]

:Exercise 1.1.1:
 - for Aalto users: set your SHELL to BASH if you have not yet done so: ``chsh -s /bin/bash`` on kosh
 - find out with *man* how to use *top* / *pstree* / *ps* to list all the running processes that belong to you
   Tip: *top* has both command line options and hot keys.

   - (*) see ``man ps`` and find out how to list a processes tree with ps, both
     all processes and only your own (but all your processes, associated with all terminals)

 - with pgrep list all bash and then zsh sessions on kosh or triton
 - log in to triton/kosh and run ``man ps``, send it to background, and ``logout``, then
   log in again. Is it still there? Play with the ``screen``, run a session , then detach it
   and log out, then log in back and get your original screen session back.
 - run ``man htop``, send it to backround, and then kill it with ``kill``. Tip: one can
   do it by background job number or by PID.
 - Imagine a use case: your current ssh session got stuck and does not response. Open another
   ssh session to the same remote host and kill the first one. Tip: ``echo $$`` gives you current
   bash PID.

   - (*) get any X Window application (firefox, xterm, etc) to run on Triton / kosh