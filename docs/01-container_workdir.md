# Preparing the container work directory

## Preparing device and partition using btrfs format

If you used the Vagrant file you should see a device vdb like below:

```
vagrant ssh
sudo fdisk -l
```

>Disk /dev/vdb: 0 MB, 197120 bytes, 385 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Now let's create a partition:

```
sudo fdisk /dev/vdb
                    
Welcome to fdisk (util-linux 2.23.2).               
                                                           
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.           
                                                                                                                                         
Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xf657fe82.
                                                       
Command (m for help): n  -------> use n for a new partition.                                     
Partition type:                                              
   p   primary (0 primary, 0 extended, 4 free)
   e   extended                          
Select (default p): p   --------> create a primary partition for the file system.                                                       
Partition number (1-4, default 1): 1   
First sector (2048-20971519, default 2048):
Using default value 2048                                    
Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519):        
Using default value 20971519                          
Partition 1 of type Linux and of 1023 MiB is set
                                                                                 
Command (m for help): w  ------> type w to write your changes.                                 
The partition table has been altered!            
                                                    
Calling ioctl() to re-read partition table.             
Syncing disks.                            
```
Now we can see our partition:

```
sudo fdisk -l

[...]
   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048    20971519     1047552   83  Linux
```
```
lsblk

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda    253:0    0   41G  0 disk 
└─vda1 253:1    0   40G  0 part /
vdb    253:16   0    1G  0 disk 
└─vdb1 253:17   0 1023M  0 part

```

Let's format it!

```
sudo mkfs.btrfs /dev/vdb1
```
You should see something like this:
```
btrfs-progs v4.9.1
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               3a53c1ce-869e-4f34-bc2e-b703b3cd0835
Node size:          16384
Sector size:        4096
Filesystem size:    1023.00MiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP              51.12MiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1  1023.00MiB  /dev/vdb1
```

Let's create a directory on the host VM for our containers:
```
sudo mkdir -p /var/lib/containers/
```

Finally let's setup our system to mount our btrfs volume on that new path.
Grab the UUID of our second partition:
```
export UUID=$(lsblk --fs /dev/vdb1 | grep -v NAME | tr -s " " | cut -d " " -f 3)
```

Let's add the second partition to fstab for it to mount automatically.
Replace the UUID below with yours or use an environment variable:
```
sudo su -c "echo 'UUID='${UUID}' /var/lib/containers/    btrfs   defaults        0 0' >> /etc/fstab"
```
Now verify your /etc/fstab file. You should see the line below in the end of your fstab file with your UUID.
```
cat /etc/fstab

UUID=5a891b40-3572-4ac6-886f-e58419cb9852 /var/lib/containers/    btrfs   defaults        0 0
```
If your file is good, finally "reload" fstab with:
```
sudo mount -a
```
Then make it private in order to isolate it from the rest of the system since we're going to create a new mount point namespace to move this one into.
```
sudo mount --make-rprivate /var/lib/containers/
```

Let's verify if it's mounted:
```
df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        40G  3.0G   35G   8% /
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G  8.5M  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
tmpfs           379M     0  379M   0% /run/user/1000
tmpfs           379M     0  379M   0% /run/user/0
/dev/vdb1      1023M   17M  904M   2% /var/lib/containers <---

or

mount
[...]
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw,relatime)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,seclabel,size=388092k,mode=700,uid=1000,gid=1000)
tmpfs on /run/user/0 type tmpfs (rw,nosuid,nodev,relatime,seclabel,size=388092k,mode=700)
/dev/vdb1 on /var/lib/containers type btrfs (rw,relatime,seclabel,space_cache,subvolid=5,subvol=/) <----
```


Now we have a btrfs partition properly mounted on /var/lib/containers/ that we can use as a work directory to build container images and create layers using the snapshot features of btrfs. Now comes the fun part:

> [Creating the container Image](02-container_image.md)

---