# Container the Hard Way

This tutorial was developed having in mind the geeky linux community and friends that would like to understand what is a container in a deeper level. That's achieved in a very simple way using just the necessary linux commands and operations that isolate an application or process using the concepts that gave birth to the container idea such as cgroups and namespaces. Using a layered file system approach we deal with volumes, images, processes, networking and more without a particular container runtime such as Docker, Containerd, Rkt, lxd/lxc etc.

I try to explain as much as I can or point to the right documentation or even man pages to make things as clear as possible. Even though, this tutorial requires a lot of linux knowlegde and was not meant for linux beginners.

 Keep in mind also that this is for educational purposes only and is not intended by any means to be used on any kind of corporate environment.

---

## Os Distribution Tested and Software Required

* Vagrant image [Centos 7.5](https://app.vagrantup.com/generic/boxes/centos7)
* Btrfs - To be used as storage driver for the writtable container layer.
* bridge-utils package to inspect bridges and interfaces

---
## How to Use This Tutorial

You can try this tutorial in any platform running Centos 7 as long as you provide a secondary volume to be used by btrfs as the container file system storage.

Some of the commands on this tutorial may cause damage to your system therefore I recommend to run them on a virtual machine separated for this purpose.

If you want to use Vagrant to run the virtual machine for you here are the requirements:

- A computer with 8 GB of ram.
- An extra 1G of disk
- libvirt/qemu installed on your linux host
- vagrant libvirt plugin also installed for your vagrant environment
- or, as an alternative, virtualbox

How to do it:
- Clone the project: `git clone git@github.com:acmenezes/container-the-hard-way.git`
- run `cd container-the-hard-way`
- run `vagrant up` and you should have a VM running for you in a couple of minutes

If you would like to know more about vagrant check Hashicorp's Vagrant docs [here](https://www.vagrantup.com/docs/index.html).

---

## Tasks:

* [Preparing the container work directory](docs/01-container_workdir.md)
* [Creating the Container Image](docs/02-container_image.md)
* [Creating the "Container Layer"](docs/03-container_layer.md)
* [Using Different namespaces](docs/04-namespaces.md)
* [Setting up the container network](docs/05-network.md)

---

## Further reading


* [Container Storage Drivers, Layers and Copy on Write](https://docs.docker.com/storage/storagedriver/)
* [Btrfs File System](https://btrfs.wiki.kernel.org/index.php/Main_Page)
* [Mount namespaces and shared subtrees](https://lwn.net/Articles/689856/)
* [Using iptables Forward and NAT rules](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/4/html/Security_Guide/s1-firewall-ipt-fwd.html)