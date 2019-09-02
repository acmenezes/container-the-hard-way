# Using Different Namespaces

A namespace is created for a process by "unsharing" it's namespace. A namespace can then be made permanent by bind-mounting the ns file to some other place.


For the moment we have our container's root file system in place. Now let's pick up a convenient program to run as the containerized application. In this case nothing better than a simple bash terminal. In oder to run it using separate namespaces we use the `unshare` command.

```
sudo unshare --mount --uts --ipc --net --pid --fork bash
```

You will see that it seems that nothing happened. But let's take advantage of the uts namespace. Read below the definition of this namespace from the man pages:

>    UTS namespaces (CLONE_NEWUTS)
UTS namespaces provide isolation of two system identifiers: the hostname and the NIS domain name.  These identifiers are set using sethostname(2) and setdomainname(2), and  can  be  retrieved  using  uname(2),  gethostname(2), and getdomainname(2).

This way we can change the hostname on this particular bash that we are running without affecting the host system and then call it again to update it. So let's try it!
```
hostname littlebox
exec bash
```
Now we have a new hostname! If you ssh into your VM again using other terminal will you see that your host system didn't change a bit! Because the changes are happening inside the new namespaces.

For now you can see that your container still has access to files inside the host file system. Let's change it making the image file system the root "/" file system.

First we need to mount our container directory on a directory placed in root. Create a directory named littlebox, create a bind mount and move it to the / path.
```
mkdir -p /littlebox
mount --bind /var/lib/containers/run/littlebox /var/lib/containers/run/littlebox
mount --move /var/lib/containers/run/littlebox /littlebox
```
Now we create an oldroot directory to place the mount point of the actual root file system on this mnt namespace. Remember, we are in isolated namespaces and the root file system can be switched by other root fs. So create the oldroot and then switch it with the new one:

From now on for practical purposes let's become root  with `sudo su`

```
cd /littlebox
mkdir oldroot
pivot_root . oldroot
```
Now verify that you have the same content in your / path as your /littlebox. The mount point of the old root is in old root! Just do `ls` on those directories and you will see it!

Finally we need to umount every old mounts from the host system. Again, go ahead, we're not doing this on the root namespace. We are doing this on the new mnt namespace.
```
umount -a
umount -l /oldroot
```
After that mount the proc file system in order to check the container processes:
```
mount -t proc none /proc
```
Then try mount. You should see a very clean scenario here.
```
mount
rootfs on / type rootfs (rw)
/dev/vdb1 on / type btrfs (rw,seclabel,relatime,space_cache,subvolid=257,subvol=/run/littlebox)
none on /proc type proc (rw,relatime)
```
Check the processes running. Again, you should see a very clean scenario with only the container processes.

```
ps faux

PID   USER     TIME  COMMAND
    1 root      0:00 bash
  106 root      0:00 ps faux
```

Now open a new terminal and ssh to the host VM again.

With the command lsns we can see the number of the inode where the namespace is "stored". This is for the current user and current process so you see vagrant and bash and all the accessible namespaces.

```
lsns

        NS TYPE  NPROCS   PID USER    COMMAND
4026531836 pid        3  3359 vagrant -bash
4026531837 user       3  3359 vagrant -bash
4026531838 uts        3  3359 vagrant -bash
4026531839 ipc        3  3359 vagrant -bash
4026531840 mnt        3  3359 vagrant -bash
4026531956 net        3  3359 vagrant -bash
```
Now to see the namespaces that our container is running we need to find it's process ID in the host system which is the ID of the unshare process and list the live namespaces under the proc file system. Take a look at the man pages of ls to understand the options Li. It's hard reference and inode number. Compare the inodes from this output with the output above. Completely different namespaces.

```
sudo ls -Lli /proc/$(pidof unshare)/ns

total 0
4026532163 -r--r--r--. 1 root root 0 Nov 22 03:17 ipc
4026532161 -r--r--r--. 1 root root 0 Nov 22 03:17 mnt
4026532166 -r--r--r--. 1 root root 0 Nov 22 03:17 net
4026531836 -r--r--r--. 1 root root 0 Nov 22 03:17 pid
4026531837 -r--r--r--. 1 root root 0 Nov 22 03:17 user
4026532162 -r--r--r--. 1 root root 0 Nov 22 03:17 uts

```
---
* [Creating the network devices for our container](05-network.md)

