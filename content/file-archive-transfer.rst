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
For Triton users the ability to transfer files to/from Triton is essential.
Same applicable to file transfer between your home workstation and kosh etc.

Several use cases::

 # transferring a file from your HOME on kosh to your home worstaion
 sftp AALTO_LOGIN@kosh.aalto.fi:file_to_copy
 
 # transferring files from Triton to your Aalto workstation
 sftp triton.aalto.fi:/scratch/work/LOGIN_NAME/some/files/* path/to/copy/to

(*) Another use case, copying to Triton, or making a directory backup with ``rsync``::

 rsync -urlptDxv --chmod=Dg+s somefile triton.aalto.fi:/scratch/work/LOGIN_NAME  # copy a file to $WRKDIR
 rsync -urlptDxv --chmod=Dg+s dir1/ triton.aalto.fi:/scratch/work/LOGINNAME/dir1/  # sync two directories

(*) Transferring and archiving your Triton data on the fly to some other place::

 # login to Triton
 cd $WRKDIR
 tar czf - path/to/dir | ssh kosh.aalto.fi 'cat > path/to/archive/dir/archive_file.tar.gz'

[Lecture notes: this session has three theory+excersise hands-ons, roughly 40+20 minutes each]

:Exercise 1.2.1:
 - Find with ``find`` all the files in your $HOME that are readable or writable by everyone

   - (*) apply ``chmod o-rwx`` to all recently found files with ``find``

 - Make a tar.gz archive of any of your directory at your HOME (or WRKDIR if on Triton), when done
   list the archive content, then append another file/directory to the existing archive.
   
   - (*) Extract only one particular file to some subdirectory from the archive
   
 - Transfer just created archive using either ``sftp`` or ``rsync``.
 
   - (*) Try ssh+tar combo to make transfer and archive on the fly.