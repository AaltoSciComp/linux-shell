File archiving and transferring
===============================

File archiving
--------------

``tar`` is the de-facto standard tool for saving many files or
directories into a single archive file.  Archive files may have
extenssions *.tar*, *.tar.gz* etc depending on compression.

::

 # create tar archive gzipped on the way
 tar -caf arhive_name.tar.gz directory_to_be_archived/
 
 # extract files
 tar -xaf archive_name.tar.gz -C path/to/directory
 
Other command line options: *r* - append files to the end of an
archive, *t* - list archive content. *f* is for the filename, and *a*
selects the compression method based on the archive file suffix (in
this example gzip, due to the .gz suffix. Without compression
files/directories are simply packed as is.

::

 # xz has better compression ratio than gzip, but is very slow
 tar -caf archive_file.tar.xz dir1/ dir2/

Individual files can be compressed directly, e.g. with ``gzip``::

 # file.gz is created, file is removed in the process.
 gzip file
 # Uncompress
 gunzip file.gz
 

Transferring files (+archiving on the fly)
------------------------------------------
As an Aalto user you may use remote directories [#aaltoremotemount]_.
Here we cover general use case, transferring from the command line.

For transferring single files or directories there is *scp*, though
for FTP like functionality there is SFTP.

::

 # transferring a file from the remote server to the current directory
 sftp LOGIN@remote.server.fi:/path/to/file_to_copy .
 
 # transferring files from your workstation to  remote server
 sftp path/to/file_to_copy LOGIN@remote.server.fi:/path/to/destination/directory

Another use case, making a directory backup with ``rsync``::

 # sync two directories
 rsync -vauW path/to/dir1/ LOGIN@remote.server.fi:/path/to/destination/dir1

(*) Transferring and archiving your data on the fly to some other place. The example
will be in covered in details later.::

 tar czf - path/to/dir | ssh LOGIN@remote.server.fi 'cat > path/to/archive/dir/archive_file.tar.gz'


Exercise 1.3
--------------

.. exercise::

 - Find with ``find`` all the files in your $HOME that are readable or writable by everyone

   - (*) apply ``chmod o-rwx`` to all recently found files with ``find``

 - Make a tar.gz archive of any of your directory, when done list the archive content
 - Extract only one particular file to a subdirectory from the archive
 - Transfer just created archive using ``sftp`` (if still time, do the same with scp and rsync).
 
   - (*) Try ssh+tar combo to make transfer and archive on the fly.


.. [#aaltoremotemount] https://scicomp.aalto.fi/aalto/remoteaccess/#remote-mounting-of-network-filesystems
