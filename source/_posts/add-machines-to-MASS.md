---
title: add machines to MASS
category: 思考
tags: 笔记
date: 2020-05-04 15:56:18
---
There are two ways to add a machine to MAAS.
* If you place the machine on a connected network, and the machine is configured to netboot, MAAS will enlist it from there. 
* If you add a machine manually, MAAS will automatically commission it. 

### How enlistment works
When MAAS enlists a machine, the first step is to contact the DHCP server, so that the machine can be assigned an IP address, which is necessary to download a kernel and initrd via TFTP.

Next, initrd mounts a Squashfs image, ephemerally, via HTTP, so that cloud-init can execute.

Finally, cloud-init runs enlistment and setup scripts.

> The enlistment scripts send the region API server information about the machine, including the architecture, MAC address and other details. The API server, in turn, stores these details in the database. This information-gathering process is known as automatic discovery.

After the enlistment process, the MAAS places the machine in the ‘Ready’ state.

Typically, the next step will be to commission the machine. 

### Add a machine manually
Enlistment can be done manually if the hardware specifications of the underlying machine are known.

On the ‘Machines’ page of the web UI, click the ‘Add hardware’ button and then select ‘Machine’.

Fill in the form and hit ‘Save machine’.

Normally, when you add a machine manually, MAAS will immediately attempt to commission the machine.
>  Note that you will need to configure the underlying machine to boot over the network, or commissioning will not happen. MAAS cannot handle this configuration for you. 