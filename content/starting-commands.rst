Starting out
============

Starter
-------

::

  du -hs * .[!.]* | sort -h
  tar czf - $d | ssh kosh.aalto.fi 'cat > public_html/$d.tar.gz && chmod a+r $d.tar.gz'
  find $d -type f \( ! -perm /g+w  -o -perm /o+w \) -exec chmod u+rwX,g+rwX,o-wx {} \;


Built-in and external commands
------------------------------

There are two types of commands:

- shell built-in: ``cd``, ``pwd``, ``echo``, ``alias``, ``bg``, ``set``, ``umask`` etc.
- external: ``ls``, ``date``, ``less``, ``lpr``, ``cat``, etc.
- some can be both: e.g. ``echo``.  Options not always the same!
- For the most part, these behave similarly, which is a good thing!
  You don't have to tell which is which.

**Hint** ``type -a`` to find what is behind the name

- **echo**: prints out ``echo something to type`` # types whatever you put after

**Disable built-in command** ``enable -n echo``, after this */usr/bin/echo*
becomes a default instead of built-in *echo*


Getting help in terminal
------------------------

Before you Google for the command examples, try::

  man command_name

Your best friend ever -- ``man`` -- collection of manuals. Type
*/search_word* for searching through the man page.  But... if it's a
builtin, you need to use ``help``.


Who/where am I
--------------

You can find info about your user (try them right away)::

  id  -or- whoami
  echo $SHELL
  hostnamectl
  pwd

For Aalto users: is your default shell a ``/bin/bash``? Login to kosh/taltta and run ``chsh -s /bin/bash``


The most common commands
------------------------

The ones we use in this tutorial, subjective::

  man, cd, pwd, echo, whoami, id, hostname, bg, fg, jobs, cat, less, ps, kill, top, pgrep,
  pstree, htop, du, cd, ls, mkdir, touch, ln, rm, cp, mv, mkdir, find, rsync, tar, scp, ssh,
  alias, set, umask, export, date, clear, head, tail, wc, grep, sort, uniq, tr, diff, killall,
  gzip, nano, xargs, chown, chmod, su, sudo, sleep, read, type, file, ping, ...
