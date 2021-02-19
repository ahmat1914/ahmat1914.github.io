---
title: What is Juju?
Category: Tech
tags: Juju
date: 2020-05-03 18:42:29
img: /images/c33e1b3bbd24b2caac6d27ef83a2afb746d2ee59.png
---

From https://juju.is/docs

---

Juju is an open source application modeling tool. It allows you to deploy, configure, scale and operate your software on public and private clouds.

### Clouds

> MAAS manages the machines and Juju manages the services running on those machines. -- From https://maas.io/docs

Juju supports  a wide variety of clouds:(cloud manages the machines,Juju manages the services)

- [Amazon AWS](https://juju.is/docs/aws-cloud) *
- [Microsoft Azure](https://juju.is/docs/azure-cloud) *
- [Google GCE](https://juju.is/docs/gce-cloud) *
- [Oracle](https://juju.is/docs/oci-cloud) *
- [Rackspace](https://juju.is/docs/rackspace-cloud) *
- [LXD](https://juju.is/docs/lxd-cloud) (local) *
- [LXD](https://juju.is/docs/lxd-cloud) (remote)
- [Kubernetes](https://juju.is/docs/k8s-cloud)
- [VMware vSphere](https://juju.is/docs/vsphere-cloud)
- [OpenStack](https://juju.is/docs/openstack-cloud)
- [MAAS](https://juju.is/docs/maas-cloud)
- [Manual](https://juju.is/docs/manual-cloud)

The exception is for a local LXD cloud; credentials are added automatically.

### Setting up

You need to install snap, which will allow you to install other software.

Install Juju locally with snap:

```bash
sudo snap install juju --classic
```

### Create a cloud on localhost

LXD manages operating system containers and makes the the “localhost” cloud available.

> LXD is a next generation system container manager. It offers a user experience similar to virtual machines but using Linux containers instead.
> It’s image based with pre-made images available for a wide number of Linux distributions and is built around a very powerful, yet pretty simple, REST API.
> — [LXD website](https://linuxcontainers.org/lxd/)

Install LXD with snap:

```bash
sudo snap install lxd
```

LXD needs to be configured for its environment:

```bash
# Use lxd help init for a list of all the options.
lxd init --auto
```

Verify that the localhost cloud is available

```bash
# Juju should have detected the presence of LXD and has added it as the localhost cloud.
juju clouds
```

### Bootstrap the controller

Juju uses an active software agent, called the controller, to manage applications.

Install the controller through a bootstrap process.

```bash
# During the bootstrap process, Juju connects with the cloud, then provision a machine to install the controller on, then install it.
# juju help bootstrap
juju bootstrap localhost overlord
```

### First workload: Hello Juju!

simple web application uses the Flask microframework to send “Hello Juju!” via HTTP.

```bash
# The charm name hello-juju is resolved into an actual charm version by contacting the Charm Store.
juju deploy hello-juju
```
