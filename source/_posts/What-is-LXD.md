---
title: What is LXD?
category: 思考
tags: 笔记
date: 2020-05-03 15:28:44
img: /images/LXD-Containers-Ubuntu-Submit-2015.png
---

From https://linuxcontainers.org/lxd/

---

LXD is a next generation system container manager. It offers a user experience similar to virtual machines but using Linux containers instead.

LXD isn't a rewrite of LXC, in fact it's building on top of LXC to provide a new, better user experience. 

LXD is written in Go, it's free software and is developed under the Apache 2 license.

The LXD project was founded and is currently led by Canonical Ltd with contributions from a range of other companies and individual contributors.

Image based.

Support for Cross-host container and image transfer

Advanced resource control (cpu, memory, network I/O, block I/O, disk usage and kernel resources)

Device passthrough (USB, GPU, unix character and block devices, NICs, disks and paths)

Network management (bridge creation and configuration, cross-host tunnels, ...)

Storage management (support for multiple storage backends, storage pools and storage volumes)

### Design

The core of LXD is a privileged daemon which exposes a REST API over a local unix socket as well as over the network (if enabled).

Clients, such as the command line tool provided with LXD itself then do everything through that REST API. It means that whether you're talking to your local host or a remote server, everything works the same way.

### Installation

```bash
snap install lxd
# or apt install lxd lxd-client
```


### Initial configuration
Before you can create containers, you need to tell LXD a little bit about your storage and network needs.

```bash
lxd init
```

### Access control
Access control for LXD is based on group membership.

* The root user as well as members of the "lxd" group can interact with the local daemon.
* Anyone with access to the LXD socket can fully control LXD, which includes the ability to attach host devices and filesystems, this should therefore only be given to users who would be trusted with root access to the host.

### Creating and using your first container

```bash
# create and start a new Ubuntu 18.04 named first
lxc launch ubuntu:18.04 first
# list all running containers
lxc list
# you can get a shell inside it with:
lxc exec first -- /bin/bash
# Or just run a command directly:
lxc exec first -- apt-get update
# To pull a file from the container, use:
lxc file pull first/etc/hosts .
# To stop the container, simply do:
lxc stop first
# And to remove it entirely:
lxc delete first
```

### Container images
LXD is image based. Containers must be created from an image and so the image store must get some images before you can do much with LXD.

There are three ways to feed that image store:

1. Use one of the the built-in image remotes
2. Use a remote LXD as an image server
3. Manually import an image tarball

**Using the built-in image remotes**

LXD comes with 3 default remotes providing images:

1. ubuntu: (for stable Ubuntu images)
2. ubuntu-daily: (for daily Ubuntu images)
3. images: (for a [bunch of other distros](https://images.linuxcontainers.org/))

```bash
lxc launch ubuntu:16.04 my-ubuntu
lxc launch ubuntu-daily:18.04 my-ubuntu-dev
lxc launch images:centos/6/amd64 my-centos
```

**Using a remote LXD as an image server**

```bash
# add a remote image server
lxc remote add my-images 1.2.3.4
# use a remote image server
lxc launch my-images:image-name your-container

# list obtained images:
lxc image list my-images:
```

**Manually importing an image**

```bash
# If you already have a lxd-compatible image file, you can import it with:
lxc image import <file> --alias my-alias
#  start a container with it:
lxc launch my-alias my-container
```

### Multiple hosts
The "lxc" command line tool can talk to multiple LXD servers and defaults to talking to the local one.

set a server work as remote server:

```bash
#  tell LXD to bind all addresses on port 8443. 
lxc config set core.https_address "[::]"
# set a trust password to be used by new clients.
lxc config set core.trust_password some-password
```

Add a remote server and use it

```bash
# This will prompt you to confirm the remote server fingerprint and then ask you for the password.
lxc remote add host-a <ip address or DNS>
# use all the same command as above but prefixing the container and images name with the remote host like:
lxc exec host-a:first -- apt-get update
```

