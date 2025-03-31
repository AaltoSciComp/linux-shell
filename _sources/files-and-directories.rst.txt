Files and directories
=====================

Files contain data.  They have a name, permissions, owner
(user+group), contents, and some other metadata.

Filenames may contain any character except '/', which is reseved as a separator between
directory and filenames. The special characters would require quotaion while dealing,
with such filenames, though it makes sence to avoid them anyway.

Path can be absolute, starts with '/' or relative, that is related to the current directory.

``ls`` is the standard way of getting information about files. By default it lists 
your current directory (i.e. *pwd*), but there are many options::

 # list directory content
 ls /usr/bin

 # list directory files including dot files (i.e. hidden ones)
 ls -A ~/directory1
 
 # list all files and directories using long format (permissions, timestamps, etc)
 ls -lA ../../directory2

Special notations and expanssions in BASH, can be used with any command::

 ./    stands for the current directory
 ../   parent directory
 ~     home directory
 *     a wildcard, replaces any character(s) or none
 ?     only one character
 []    a group of characters, like [abc]
 [!]   same as above but negated, i.e. all but those, like [!a-zA-Z]
 {ab,cd,xyz}  expands by BASH as 'ab cd xyz'
 {0..9}      expands as '0 1 2 3 4 5 6 7 8 9'
 
For the quotation::

 '', "", \

Quotation matters ``ls 'file name'`` vs ``ls file name`` or ``echo "$USER"`` vs ``echo '$USER'``


BASH first expand the expanssions and substitute the wildcards, and then
execute the command. Could be as complex as::

 ls -l ~/[!abc]???/dir{123,456}/filename*.{1..9}.txt

There are a variety of commands to manipulate files/directories::

 cd, mkdir, cp, cp -r, rm, rm -r, mv, ln, touch
 
For file/directory meta information or content type::

 ls, stat, file, type -a

Note that ``cd`` is a shell builtin which change's the shell's own
working directory.  This is the base from which all other commands
work: ``ls`` by default tells you the current directory.  ``.`` is the
current directory, ``..`` is the parent directory, ``~`` is your HOME.  This is
inherited to other commands you run. ``cd`` with no options drops your to your $HOME.

::

 # copy a directory preserving all the metadata to two levels up
 cp -a dir1/ ../../

 # move all files with the names like filename1.txt, filename_abc.txt etc to dir2/
 mv filename*.txt dir2/
 
 # remove a directories/files in the current dir without asking for the confirmation
 rm -rf dir2/ dir1/ filename*
 
 # create an empty file if doesn't exist or update its access/modification time
 touch filename
 
 # create several directories at once
 mkdir dir3 dir4 dir5
 # -or-
 mkdir dir{3,4,5}
 
 # make a link to a target file (hard link by default, -s for symlinks)
 ln target_file ../link_name


**Discover other ls features** ``ls -lX``, ``ls -ltr``, ``ls -Q``

You may also find useful ``rename`` utility implemented by Larry Wall.


File/directory permissions
--------------------------
- Permissions are one of the types of file metadata.
- They tell you if you can *read* a file, *write* a file, and
  *execute a file/list directory*
- Each of these for both *user*, *group*, and *others*
- Here is a typical permission bits for a file: ``-rw-r--r--``
- In general, it is ``rwxrwxrwx`` -- read, write, execute/search for
  user, group, others respectively
- ``ls -l`` gives you details on files.

Modifying permissions: the easy part
------------------------------------

chmod/chown is what will work on all filesystems::

 chmod u+rwx,g-rwx,o-rwx <files>   # u=user, g=group, o=others, a=all
 # -or-
 chmod 700 <files>   # r=4, w=2, x=1
 
 # recursive, changing all the subdirectories and files at once
 chmod -R <perm> <directory>

 # changing group ownership (you must be a group member)
 chgrp group_name <file or directory>

Extra permission bits:

- s-bit:  setuid/setgid bit, preserves user and/or group IDs.
- t-bit: sticky bit, for directories it prevents from removing file by
  another user (example */tmp*)

Setting default access permissions: add to *.bashrc* ``umask 027``
[#umask]_.  The ``umask`` is what permissions are *removed* from any newly
created file by default.  So ``umask 027`` means "by default,
g-w,o-rwx any newly created files".  It's not really changing the
permissions, just the default the operating system will create with.


**Hint:**
even though file has a read access the top directory must be
searchable before external user or group will be able to access
it. Sometimes on Triton, people do ``chmod -R o-rwx $WRKDIR; chmod o+x
$WRKDIR``.  Execute (``x``) without read (``r``) means that you can
access files inside if you know the exact name, but not list the
directory.  The permissions of the files themselves still matter.


Modifying permissions: advanced (*)
-----------------------------------
Access Control Lists (ACLs) are advanced access permissions.  They
don't work everywhere, for example mostly do no work on NFS
mounted directories.  They are otherwise supported on ext4, lustre,
etc (thus works on Triton $WRKDIR).

* In "normal" unix, files have only "owner" and "group", and permissions
  for owner/group/others.  This can be rather limiting.
* Access control lists (ACLS) are an extension that allows an
  arbitrary number of users and groups to have access rights to
  files.  The basic concept is that you have: 
* ACLs don't show up in normal ``ls -l`` output, but there is an extra
  plus sign: ``-rw-rwxr--+``.  ACLs generally work well, but there are
  some programs that won't preserve them when you copy/move files, etc.
* POSIX (unx) ACLs are controlled with ``getfacl`` and ``setfacl``

  - Allow read access for a user ``setfacl -m u:<user>:r <file_or_dir>``
  - Allow read/write access for a group ``setfacl -m g:<group>:rw <file_or_dir>``
  - Revoke granted access ``setfacl -x u:<user> <file_or_dir>``
  - See current stage ``getfacl <file_or_dir>``

**File managers** on Triton we have installed Midnight Commander -- ``mc``

**Advanced file status** to get file meta info ``stat <file_or_dir>``



Exercise 1.2
------------

.. exercise::

 - mkdir in the current direcotory, cd there and ``touch`` a file.
   Rename it. Make a copy and then remove the original.  What does ``touch`` do?
 - list all files in /usr/bin and /usr/sbin that start with non-letter characters with
   one ``ls`` command
 - list with ``ls`` dot files/directories only (by default it
   lists all files/directories but not those that begin with ``.``).
   "dotfiles" are a convention where filenames that begin with ``.``
   such as ``.bashrc`` are considered "hidden".
 - Explore ``stat file`` output. What metadata do you find?  Try
   to stat files of different types (a regular file, directory, link,
   special device in /dev)
 - create a directory, use ``chmod`` to allow user and any group members
   full access and no access for others
 - (*) change that directory group ownership with ``chown`` or ``chgrp`` (any group that you
   belong to is fine), set s-bit for the group and
   apply t-bit to a directory, check that the upper directory has *o+x* bit set: now you should
   have a private working space for your group. Tip: see groups that you are a member of ``id -Gn``
 - ``ls -ld`` tells you that directory has permissions ``rwxr-Sr--``. Do group members have
   access there?
 - (*) create a directory (in /tmp if you are on a server with ),
   use ``setfacl`` to set its permissions so that only you and some
   user/group of your choice would have access to it.
 - (*) create a directory and a subdirectory in it and set their permissions to 700 with one command.


.. [#umask] https://www.computerhope.com/unix/uumask.htm
