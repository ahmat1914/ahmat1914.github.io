---
title: What is MASS?
category: 思考
tags: 笔记
date: 2020-05-03 18:24:09
img: /images/c7ab255f51a3931815b5beeaf19119a42ea0e9e4.png
---
From https://maas.io/docs

---

### What is MASS ?

MAAS is **Metal As A Service**,  converts bare-metal servers into cloud instances of virtual machines, as if they are instances hosted in a public cloud like AWS. ( KVM guests can even act as MAAS machines if they boot from the network via PXE.)

You can discover, commission, deploy, and dynamically reconfigure a large network of individual units. 

MAAS can act as a standalone PXE/preseed service or integrate with other technologies.It works exceptionally well with [Juju](https://jaas.ai/docs/maas-cloud), the service and model management tool. MAAS manages the machines and Juju manages the services running on those machines.

### What MAAS offers?

MAAS can manage a large number of physical machines by merging them into user-defined resource pools. 

MAAS integrates all the tools you need into a smooth system-management experience. It includes:

- web UI (optimised for mobile devices)
- Ubuntu, CentOS, Windows, and RHEL installation support
- open-source IP address management (IPAM)
- full API/CLI support
- high availability (optional)
- IPv6 support
- inventory of components
- DHCP and DNS for other devices on the network
- DHCP relay integration
- VLAN and fabric support
- NTP for the entire infrastructure
- hardware testing
- composable hardware support

These tools can be controlled from a responsive [web UI](https://maas.io/docs/web-ui) or a [CLI](https://maas.io/docs/maas-cli) driven by a REST API. 

### Colocation of key components

MAAS relies on two key components: the *region controller* and the *rack controller*.(In essence, rack controllers manage racks, while the region controller manages the data centre. )

* The region controller handles operator requests; 
* the rack controller provides high-bandwidth services to multiple racks. 

> We generally recommended installing both controllers on the same system. The default MAAS install delivers this colocated configuration automatically. This all-in-one solution also provides [DHCP](https://maas.io/docs/dhcp).

### How MAAS works?

When you [add a new machine](https://maas.io/docs/add-nodes#heading--add-a-node-manually) to MAAS, or elect to add a machine that MAAS has [enlisted](https://maas.io/docs/add-nodes#heading--enlistment), MAAS [commissions](https://maas.io/docs/commission-nodes) it for service and adds it to the pool. At that point, the machine is ready for use. MAAS keeps things simple, marking machines as “New,” “Commissioning,” “Ready,” and so on.

There are two ways to add a machine to MAAS. Assuming it’s on the network and capable of PXE-booting, you can add it explicitly – or MAAS can simply discover it when you turn it on.

Enlistment just means that MAAS discovers a machine when you turn it on, and presented to the MAAS administrator, so that they can choose whether or not to commission it.Machines that have only enlisted will show up in the machine list as “New.”

Commissioning means that MAAS has successfully booted the machine, scanned and recorded its resources, and prepared it for eventual deploymen.Machines that you explicitly add are automatically commissioned. MAAS marks a successfully-commissioned machine as “Ready” in the machine list.

MAAS controls machines through IPMI (or another BMC). 

> MAAS overwrites the machine’s disk space with your chosen, pre-cached OS images.

MAAS users allocate (“acquire”) machines for use when needed. The web UI also allows you to acquire machines manually, such as when you are reserving specific hardware for certain users. You can remotely access and customise the installed operating system via SSH.

Note that [Juju](https://jaas.ai/docs/maas-cloud) is designed to work with MAAS. MAAS becomes a backend Juju resource pool with all functionality fully available. For instance, if Juju removes a machine, then MAAS will release that machine to the pool. With Juju, MAAS can become an integral part of your data centre strategy and operations.