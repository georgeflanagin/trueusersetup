# trueusersetup

## The trueuser setup

TrueUser is a small NAS, with three 2TB disks in a Raid Z1 (RAID5) configuration. There is only one 
pool, and the dataset named `/mnt/usrlocal` is exported, read only, and only available on the UR
network. The NFS sharing is set up so that clients can mount any directory within this dataset without
mounting the "whole thing," thereby allowing clients to pick and choose.

The Linux 8 workstations mount `/mnt/usrlocal/8` as their `/usr/local/chem.sw`, so that they see the Linux 8
software. Here is the relevant line in the client computers' `/etc/fstab`:

```
141.166.186.35:/mnt/usrlocal/8  /usr/local/chem.sw  nfs     ro,nosuid,nofail,_netdev 0 0
```

For those unfamiliar with the syntax for mounting NFS shares, the terms have these meanings:

```
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

Alongside `/mnt/usrlocal/8` is `/mnt/usrlocal/9`. There are two types of programs that need to be 
made available. 

### Linux 9, compiled binaries

For these programs, we follow the same process used for Linux 8:

1. Choose a Linux 9 computer as the build environment.
2. Install the development tools, and build the code.
3. Use `scp` to move the files from the build environment to the NAS.
4. On the NAS, `chown nobody:nogroup` on the new software.

