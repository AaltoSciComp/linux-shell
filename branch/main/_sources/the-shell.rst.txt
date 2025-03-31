Basic shell
===========

First touch: getting a BASH shell
---------------------------------

Set yourself up with a BASH shell.  Connect to a server or open on your own computer.
Examples and demos given during the lecture are done on a native Linux BASH on either Ubuntu or CentOS,
though should work on all other Linux installations and Git BASH on Windows.

- Linux and Mac users: just open a terminal window

    - Ubuntu users hint: Ctrl + Alt + t
  
- Windows users:

    - (Priority #1) SSH to a remote native Linux installation. For aalto users [#aaltoremoteaccess]_.

      - Use built-in SSH client: open a Command Prompt (cmd) and launch a command 'ssh loginname@remote.server.name'
      - If no built-in client, install PuTTY [#putty]_.

    - (For easy testing) Git BASH [#gitbash]_  // Misses man pages, some utilities.
    - -or- VDI if such service available. Aalto user, see Remote desktop at [#aaltoremoteaccess]_.
    - -or- Windows Subsystem for Linux (WSL 2) and Ubuntu on Windows [#ubuntuwindows]_.
    - -or- Cygwin [#cygwin]_.


About the Linux Shell
---------------------

- A *shell* is what you get when your terminal window is open. It is a
  command-line (CLI), an interface that interpreters and executes the
  commands.
- The name comes from being a "shell" (layer) around the operating
  system.  It connects and binds all programs together.
- This is the basic, raw method of using UNIX-like systems.  It may
  not be used everyday, but it's really good (necessary) for any type
  of automation and scripting - as is often needed in science, when
  connecting pieces together, or when using Triton.
- There are multiple shells.  This talk is about `bash
  <https://en.wikipedia.org/wiki/Bash_(Unix_shell)>`__, which is the
  most common one.  `zsh <https://en.wikipedia.org/wiki/Z_shell>`__ is
  another common shell which is somewhat similar but has some more
  powerful features.  `tcsh <https://en.wikipedia.org/wiki/Tcsh>`__ is
  another shell from a completely different family (the csh family),
  which has quite different syntax.
- ``bash`` is a "Bourne shell": the "bourne-again shell".  An open source
  version of original Bourne shell.
- It may not be obvious, but the concepts here also apply to Windows
  programs and will help you understand them.  They also apply more
  directly to Mac programs, because Mac is unix under the hood.


Basic shell operation
---------------------

- You type things on the screen (standard input or stdin).  The shell
  uses this to make a command.
- The shell takes the command, splits it into words, does a lot more
  preprocessing, and then runs it.
- When the command runs, the keyboard (still standard input) goes to
  the process, output (standard output) goes to the screen.


.. [#aaltoremoteaccess] https://scicomp.aalto.fi/aalto/remoteaccess/
.. [#putty] https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
.. [#gitbash] https://gitforwindows.org/
.. [#ubuntuwindows] https://www.microsoft.com/en-us/p/ubuntu/9nblggh4msv6
.. [#cygwin] https://www.cygwin.com/

