mkchroot and rmchroot utilities

---------

Usage:

    mkchroot [options] /path/to/chroot < object.spec

Create and populate a chroot directory. Options are:

    -a       - add to an existing chroot
    -d       - print debug info
    -u umask - set the umask, default is 022
    -z size  - max memory size of a new chroot, default is 32K (see tmpfs size
               option in man mount)

If -a, then the specified chroot directory must already exist, created by a
previous invocation of mkchroot, and -z is ignored.

Otherwise the specified directory must not exist. The directory is created, and
a tmpfs of specified maximum size is mounted on it.

Object specifications are then read from stdin. #comments and blank lines
ignored. A valid line is in the form:

    [ [ perm [ ... perm ] ] : ] /path/to/object [ ... /path/to/object ]

That is, it names one or more objects in the root that will be imported to the
same path in the chroot directory. 

Absolute path names are required. Globs are supported and will expand into all
matching names. 

Allowed objects are device nodes, symlinks, files, or directories. 

Device nodes and symlinks are copied verbatim. 

Files and directories are bind mounted. By default, bind mounts are read-only,
noexec, and nodev.  However if the line contains a ":", then tokens to the left
are treated as permission flags, presence of "rw", "exec", and/or "dev" will
disable the corresponding mount options for all objects on the line.

(If an object name contains ":" for some reason, then the line must start with
":" even if no special permissions are required.)

Exit status is 0 if the chroot was created and all objects imported. Otherwise
status is non-zero and and the partially constructed chroot may be left in
place (use rmchroot to delete it).

---------

Usage: 
    
    rmchroot /path/to/chroot

Delete a chroot directory created by mkchroot. 

If the specified directory has a tmpfs mounted on it, then search /proc/mounts
for all objects mounted into that directory and unmount them in descending
order of path length, ending with the containing tmpfs. Then remove the
directory. 

Exit status is 0 if the chroot was removed successfully. Otherwise status is
non-zero and and the partially deconstructed chroot may be left in place.
