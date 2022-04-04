How to make things faster: hotkeys
==================================

- Is it annoying to have to type everything in the shell?  No, because
  we have hotkeys.  In fact, it can become much more efficient and
  powerful to use the shell.
- Most important key: **TAB**: autocomplete.  You should never be
  typing full filenames or command names.  TAB can complete almost anything

Common hotkeys:

- TAB -- autocomlpetion
- Home ``or`` Ctrl-a -- start of the command line
- End ``or`` Ctrl-e -- end
- Ctrl-left/right arrows ``or`` Alt-b/Alt-f  - moving by one word there and back
- up/down arrows -- command history
- Ctrl-l -- clear the screen
- Ctrl-Shift-c -- copy
- Ctrl-Shift-v -- paste
- Ctrl-Shift--  -- undo the last changes on cli
- Alt-r -- undo all changes made to this line
- Ctrl-r -- command history search: backward (hit Ctrl-r, then start typing the search word, hit Ctrl-r again to go through commands that have the search word in it)
- Ctrl-s  -- search command history furtherword (for this to work one needs to disable default suspend keys ``stty -ixon``)
- Ctrl-u  -- remove beginning of the line, from cursor
- Ctrl-k -- remove end of the line, from cursor
- Ctrl-w -- remove previous word

**inputrc** Check */etc/inpurc* for some default key bindings, more can be defined *~/.inputrc* (left as a home exercise)

**CDPATH** helps changing directories faster. When you type ``cd dirname``, the shell tries to go
to one of the local subdirectories and if it is not found shell will try the same command from every
directory listed in the *$CDPATH*.

::

 export CDPATH=$HOME:$WRKDIR:$WRKDIR/project