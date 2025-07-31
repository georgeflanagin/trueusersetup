# trueusersetup

## The trueuser setup

TrueUser is a small NAS, with three 2TB disks in a Raid Z1 (RAID5) configuration using the ZFS file system.
There is only one 
pool, and the dataset named `/mnt/usrlocal` is exported, read only, and only available on the UR
network. The NFS sharing is set up so that clients can mount any directory within this dataset without
mounting the "whole thing," thereby allowing clients to pick and choose.

The Linux 8 workstations mount `/mnt/usrlocal/8` as their `/usr/local/chem.sw`, so that they see the Linux 8
software. Here is the relevant line in the client computers' `/etc/fstab`:

```bash
141.166.186.35:/mnt/usrlocal/8  /usr/local/chem.sw  nfs     ro,nosuid,nofail,_netdev 0 0
```

For those unfamiliar with the syntax for mounting NFS shares, the terms have these meanings:

```bash
141.166.186.35:/mnt/usrlocal/8  <<-- the location of the directory
/usr/local/chem.sw              <<-- its name in the client computer's file system
nfs                             <<-- the mechanism for mounting it.
ro,nosuid,nofail,_netdev        <<-- read only,
                                     no suid bits
                                     do not stop the boot if it is unavailable
                                     do not try to mount until the network is available.
0 0                                  do not back it up or try to recover it with fschk.
```

## Migration to Rocky Linux 9

### Copying the directories

Alongside `/mnt/usrlocal/8` is `/mnt/usrlocal/9`. The directory structure of `/mnt/usrlocal/8` (but not
the files!) has been cloned onto `/mnt/usrlocal/9` with this command:

```bash
cd /mnt/usrlocal/8
find . -type d -exec mkdir -p ../9/{} \;
```

This process ensures that scripts referencing locations in Linux 8 environment will work unmodified in
the Linux 9 environment, at least for the directory names and the environment variables that contain them.

### Migrating the files

Copying all the files from the 
/8 to the /9 directory creates two problems:

1. It takes up a *lot* of space.
2. Bug fixes and all other changes to the scripts that reside in directories like `/mnt/usrlocal/8/etc/modulefiles` will require that the change be repeated in the /9 directory, introducing the opportunity to make mistakes or to entirely forget that it needs to be done.

The solution for this problem is to link the files in /9 to the files in /8. However NFS does not
allow symbolic links to resolve to a file or directory that resides in a non-mounted directory. Thus,
we need to use the somewhat less common non-symbolic links, often called "hard links," although that
terms is not technically correct. They are just "links."

This command: 

```bash
ln /some/directory/somefile.py /some/other/directory/differentname.py
```

Creates a new directory entry that points to the same sectors on disk, effectively giving the
data two different names. Of course, if you need to link a few hundred thousand files, this 
approach is impractical. Fortunately, the Swiss Army Knife of the file system, `rsync` comes
to our rescue:

```bash
rsync -av --progress --ignore-existing --link-dest=/mnt/usrlocal/8 /mnt/usrlocal/8/ /mnt/usrlocal/9/
```
And here is an explanation of the frequently bewildering options of `rsync`:

```bash
-a                              archive mode traverses the directory tree
v                               verbosity let's us observe 
--progress                      prints a progress bar for entertainment
--ignore-existing               will not overwrite any file in the destination
--link-dest=/mnt/usrlocal/8     creates links to the corresponding files rather than copies
/mnt/usrlocal/8/                the source .. everything in 8/
/mnt/usrlocal/9/                the destination
```

Files that have more than one link (i.e., a name in both the 8/ and 9/ directories) will show up with
a `2` in the link count column when using the `ls -l` command:

```
-rwxr-xr-x  2 nobody  nogroup  uarch    231115 Apr  4  2016 h5dump*
```

whereas files that are not linked across the 8/ and 9/ environments have only only `1` link:

```
-rwxrwxr-x. 1 nobody nogroup   8998920 Aug 21  2024 sander
```

Although it is somewhat impractical considering the number of files on the NAS, you can identify all the files
that are linked between/across the two environments with this command:

```bash
find /mnt/usrlocal/9 -type f -links +1
```

... further proof that `find` has an option for almost every file characteristic.


Although it is somewhat impractical considering the number of files in

### Linux 9, compiled binaries

For these programs, we follow the same process used for Linux 8:

1. Choose a Linux 9 computer as the build environment.
2. Install the development tools, and build the code.
3. Use `scp` to move the files from the build environment to the NAS.
4. On the NAS, `chown nobody:nogroup` on the new software to ensure availability and correct permissions.

## Consderations

This system does save a great deal of space, and from the standpoing that many of files being
served to the workstations are identical on Linux 8 and Linux 9, caching is improved
because the NAS is aware when a file is identical, and just has two names. 

However, it is easy to think of the two directory hierarchies as being syncronized, and this is
false. Adding *new* items to the 8/ directory or the 9/ directory will not automatically create
a copy in the other hierarchy. OTOH, edits to an existing file (examples: login scripts, module files)
are visible in both directories because there is only one underlying file.

