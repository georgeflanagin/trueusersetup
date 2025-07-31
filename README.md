# trueusersetup

## The trueuser setup

TrueUser is a small NAS, with three 2TB disks in a Raid Z1 (RAID5) configuration. There is only one 
pool, and the dataset named `/mnt/usrlocal` is exported, read only, and only available on the UR
network. 

The Linux 8 workstations mount `/mnt/usrlocal/8` as their `/usr/local/chem.sw`, so that they see the Linux 8
software. Here is the relevant line in the client computers' `/etc/fstab`:

```
141.166.186.35:/mnt/usrlocal/8 /usr/local/chem.sw nfs auto,nofail 0 0
```
