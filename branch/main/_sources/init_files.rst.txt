Initialization files and file editing
=====================================

Initialization files and configuration
--------------------------------------
- When the shell first starts (when you login), it reads some files.
  These are normal shell files, and it evaluates normal shell commands
  to set configuration.
- You can always test things in your own shell and see if it works
  before putting it in the config files.  Highly recommended!
- You customize your environment means setting or expanding aliases,
  variables, functions.
- The config files are:

  - ``.bashrc`` (when SSH) and
  - ``.bash_profile`` (interactive login to a workstation)
  - they are often a symlink from one to another
  
- To get an idea how complicated .bashrc can be take a look at <https://www.tldp.org/LDP/abs/html/sample-bashrc.html>


One of the things to play with: command line prompt defined in PS1 [#ps1]_

::

 PS1="[\d \t \u@\h:\w ] $ "

For special characters see PROMPTING at ``man bash``. To make it
permanent, should be added to *.bashrc* like ``export PS1``.


Creating/editing/viewing file
------------------------------
* A *text editor* edits files as ASCII.  These are your best friend.
  In fact, text files are your best friend: rawest, most efficient,
  longest-lasting way of storing data.
* "pager" is a generic term for things that view files or data.

Linux command line *text editors* like:

- *nano* - simplest
- *vim* - minimal.  To save&quit, ``ESC :wq``
- *emacs* - or the simplest one *nano*.  To save&quit: ``Ctrl-x
  Ctrl-c``

To view contents of a file in a scrollable fashion: ``less``

Quick look at the text file ``cat filename.txt`` (dumps everything to
screen- beware of non-text binary files or large files!)

Other quick ways to add something to a file (no need for an editor)

``echo 'Some sentence, or whatever else 1234567!-+>$#' > filename.txt``

``cat > filename2.txt`` to finish typing and write written to the file, press enter, then Ctrl-d.

**The best text viewer ever** ``less -S``  (to open a file in your EDITOR, hit *v*, to search through type */search_word*)

**Watching files while they grow** ``tail -n 0 -f <file>``

Try: add above mentioned ``export PS1`` to *.bashrc*. Remember ``source .bashrc`` to enable changes


Exercise 1.5
--------------

.. exercise::

 - link *.bash_profile* to *.bashrc*. Tip: see ``ln`` command from the previous session.
 - add ``umask 027`` to *.bashrc*, try creating files. Tip: ``umask -S`` prints your current setting.
 - customize a prompt ``$PS1`` and add it to your *.bashrc*, make sure is has
   a current directory name and the hostname in it in the format *hostname:/path/to/current/dir*.
   Hint: save the original PS1 like ``oldPS1=$PS1`` to be able to recover it any time.


.. [#ps1] https://www.ibm.com/developerworks/linux/library/l-tip-prompt/
