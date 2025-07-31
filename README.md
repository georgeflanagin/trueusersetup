# trueusersetup

## The trueuser setup

TrueUser is a small NAS, with three 2TB disks in a Raid Z1 (RAID5) configuration. There is only one 
pool, and the dataset named `/mnt/usrlocal` is exported, read only, and only available on the UR
network. 

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

