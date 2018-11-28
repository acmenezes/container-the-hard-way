# Container the Hard Way

This tutorial aims to show the "behind the scenes" of a container run time. This is a walk through of linux commands and operations necessary to isolate an application using the concepts that gave birth to the container idea such as cgroups and namespaces on a layered file system approach. Here you will deal with volumes, images, processes, networking and more without using a particular container run time such as Docker, Containerd or Rkt. This is for educational purposes only and doesn't intend to give you everything needed to write a container runtime.

> Here I try to explain as much as I can or point to the right documentation or even man pages to make things as clear as possible. Even though, this tutorial requires a lot of linux knowlegde and was not meant for linux beginners.

---

## Target Audience

Everyone that wants to have a deeper knowledge on how containers are built and run from the system's perspective and also for those who want to master the linux behind container runtime applications ir order to contribute and develop new features after.

---

## Os Distribution Tested and Software Required

* Vagrant image [Centos 7.5](https://app.vagrantup.com/generic/boxes/centos7)
* Btrfs - To be used as storage driver for the writtable container layer.
* bridge-utils package to build the container bridge by hand

---
## How to Use This Tutorial




## Tasks:

* [Prerequisites](docs/00-prereqs.md)
* [Installing needed packages](docs/01-packages.md)
* [Preparing the container work directory](docs/02-container_workdir.md)
* [Creating the Container Image](docs/03-container_image.md)
* [Creating the "Container Layer"](docs/04-container_layer.md)
* [Using Different namespaces](docs/05-namespaces.md)
* [Setting up the container network](docs/06-network.md)
* [Setting up cgroups for the new container](docs/01-cgroups.md)
* [Setting up devices for the new container]()
* [Setting up capabilities for the new container]()
* [Setting up Security for the new container]()
* [wrapping everything up]()