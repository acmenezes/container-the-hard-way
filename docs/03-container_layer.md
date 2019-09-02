# Creating the Container Layer

Now that we have the file system in our image directory let's create a snapshot of this image for the container layer and call our new container instance littlebox.

First let's create a directory for our running containers:
```
sudo mkdir -p /var/lib/containers/run
```
Now let's create the snapshot of our image inside this new directory:
```
sudo btrfs subvol snapshot /var/lib/containers/img/alpine/ /var/lib/containers/run/littlebox
```
Let's check our snapshot directory:

```
sudo ls -l /var/lib/containers/run/littlebox/

lrwxrwxrwx. 1 root root    7 Nov 21 03:23 bin -> usr/bin
dr-xr-xr-x. 1 root root    0 Apr 11  2018 boot
drwxr-xr-x. 1 root root    8 Nov 21 03:23 dev
drwxr-xr-x. 1 root root 1242 Nov 21 03:23 etc
drwxr-xr-x. 1 root root    0 Apr 11  2018 home
lrwxrwxrwx. 1 root root    7 Nov 21 03:23 lib -> usr/lib
lrwxrwxrwx. 1 root root    9 Nov 21 03:23 lib64 -> usr/lib64
drwxr-xr-x. 1 root root    0 Apr 11  2018 media
drwxr-xr-x. 1 root root    0 Apr 11  2018 mnt
drwxr-xr-x. 1 root root    0 Apr 11  2018 opt
dr-xr-xr-x. 1 root root    0 Apr 11  2018 proc
dr-xr-x---. 1 root root    0 Apr 11  2018 root
drwxr-xr-x. 1 root root    0 Nov 21 03:38 run
lrwxrwxrwx. 1 root root    8 Nov 21 03:23 sbin -> usr/sbin
drwxr-xr-x. 1 root root    0 Apr 11  2018 srv
dr-xr-xr-x. 1 root root    0 Apr 11  2018 sys
drwxrwxrwt. 1 root root    0 Apr 11  2018 tmp
drwxr-xr-x. 1 root root  106 Nov 21 03:23 usr
drwxr-xr-x. 1 root root  160 Nov 21 03:23 var
```

Here we are actually seeing the same data that is in the img/alpine directory. We're are just reading and not writing anything yet. No changes so far. But to follow our container during this process let's place a file there!

```
sudo su -c 'echo "CONTAINER DATA" >> /var/lib/containers/run/littlebox/THIS_IS_OUR_CONTAINER_DATA'
```

Let's copy the container path to a variable:
```
export CONTAINER=/var/lib/containers/run/littlebox
```
And finally do little test with chroot using alpine's sh:
```
sudo chroot ${CONTAINER} sh
/ #

ls -l
-rw-r--r--    1 root     root            15 Nov 21 20:51 THIS_IS_OUR_CONTAINER_DATA
drwxr-xr-x    1 root     root           850 Sep 11 13:58 bin
drwxr-xr-x    1 root     root            20 Sep 11 13:58 dev
drwxr-xr-x    1 root     root           490 Sep 11 13:58 etc
drwxr-xr-x    1 root     root             0 Sep 11 13:58 home
drwxr-xr-x    1 root     root           336 Sep 11 13:58 lib
drwxr-xr-x    1 root     root            28 Sep 11 13:58 media
drwxr-xr-x    1 root     root             0 Sep 11 13:58 mnt
dr-xr-xr-x    1 root     root             0 Sep 11 13:58 proc
drwx------    1 root     root            24 Nov 21 20:52 root
drwxr-xr-x    1 root     root             0 Sep 11 13:58 run
drwxr-xr-x    1 root     root           792 Sep 11 13:58 sbin
drwxr-xr-x    1 root     root             0 Sep 11 13:58 srv
drwxr-xr-x    1 root     root             0 Sep 11 13:58 sys
drwxrwxrwt    1 root     root             0 Sep 11 13:58 tmp
drwxr-xr-x    1 root     root            40 Sep 11 13:58 usr
drwxr-xr-x    1 root     root            78 Sep 11 13:58 var

```
Look that you can run Alpine exclusive binaries from it's shell such as apk:
```
apk --help

apk-tools 2.10.1, compiled for x86_64.

Installing and removing packages:
  add       Add PACKAGEs to 'world' and install (or upgrade) them, while ensuring that all dependencies are met
  del       Remove PACKAGEs from 'world' and uninstall them

System maintenance:
  fix       Repair package or upgrade it without modifying main dependencies
  update    Update repository indexes from all remote repositories
  upgrade   Upgrade currently installed packages to match repositories
  cache     Download missing PACKAGEs to cache and/or delete unneeded files from cache

Querying information about packages:
  info      Give detailed information about PACKAGEs or repositories
  list      List packages by PATTERN and other criteria
  dot       Generate graphviz graphs
  policy    Show repository policy for packages

Repository maintenance:
  index     Create repository index file from FILEs
  fetch     Download PACKAGEs from global repositories to a local directory
  verify    Verify package integrity and signature
  manifest  Show checksums of package contents

Use apk <command> --help for command-specific help.
Use apk --help --verbose for a full command listing.

This apk has coffee making abilities.
```

Let's get back and continue.
```
exit
```

Next step:

* [Using Different namespaces](04-namespaces.md)



