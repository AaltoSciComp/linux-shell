Starting out
============

Starter
-------

::

  du -hs * .[!.]* | sort -h
  tar czf - $d | ssh kosh.aalto.fi 'cat > public_html/$d.tar.gz && chmod a+r $d.tar.gz'
  find $d -type f \( ! -perm /g+w  -o -perm /o+w \) -exec chmod u+rwX,g+rwX,o-wx {} \;


Getting help in terminal
------------------------

Before you Google for the command examples, try::

  man command_name

Your best friend ever -- ``man`` -- collection of manuals. Type
*/search_word* for searching through the man page.  But... if it's a
builtin, you need to use ``help``.


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
