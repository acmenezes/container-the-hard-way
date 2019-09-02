# Creating the container Image

## Create a btrfs subvolume

A subvolume is an independent part of the file system capable of sharing it's file extents which gives us the possibility of taking snapshots of it. That is what enables us to create new layers based on top of others. Check the man page [`man btrfs-subvolume`](https://btrfs.wiki.kernel.org/index.php/Manpage/btrfs-subvolume) to know more about btrfs subvolumes.


Let's create our image directory as a subvolume inside our work directory and set our environment variables:
```
export WORKDIR=/var/lib/containers
sudo mkdir -p ${WORKDIR}/img
export IMAGEDIR=${WORKDIR}/img/alpine
sudo btrfs subvolume create ${IMAGEDIR}
```
Now let's check our new subvolume:
```
sudo btrfs subvol list -p /var/lib/containers/img

ID 256 gen 7 parent 5 top level 5 path img/alpine
```
---
## Let's download and extract an Alpine Linux image

Here we could install any filesystem we want. For practical purposes I'm using one of the most popular container images, the alpine image. Keep in mind that you need to check the version and the url first from the alpine linux website. Below is the version that I got at the creation of this tutorial.

```
curl -s http://dl-cdn.alpinelinux.org/alpine/v3.8/releases/x86_64/alpine-minirootfs-3.8.1-x86_64.tar.gz | sudo tar xvzf - -C /var/lib/containers/img/alpine/
```
Let's check it!

```
sudo ls -l /var/lib/containers/img/alpine/

drwxr-xr-x. 1 root root 850 Sep 11 13:58 bin
drwxr-xr-x. 1 root root  20 Sep 11 13:58 dev
drwxr-xr-x. 1 root root 490 Sep 11 13:58 etc
drwxr-xr-x. 1 root root   0 Sep 11 13:58 home
drwxr-xr-x. 1 root root 336 Sep 11 13:58 lib
drwxr-xr-x. 1 root root  28 Sep 11 13:58 media
drwxr-xr-x. 1 root root   0 Sep 11 13:58 mnt
dr-xr-xr-x. 1 root root   0 Sep 11 13:58 proc
drwx------. 1 root root   0 Sep 11 13:58 root
drwxr-xr-x. 1 root root   0 Sep 11 13:58 run
drwxr-xr-x. 1 root root 792 Sep 11 13:58 sbin
drwxr-xr-x. 1 root root   0 Sep 11 13:58 srv
drwxr-xr-x. 1 root root   0 Sep 11 13:58 sys
drwxrwxrwt. 1 root root   0 Sep 11 13:58 tmp
drwxr-xr-x. 1 root root  40 Sep 11 13:58 usr
drwxr-xr-x. 1 root root  78 Sep 11 13:58 var
```
Now Let's use this image to build the container layer:

* [Creating the "Container Layer"](03-container_layer.md)
