---
title: Concepts
weight: 100
description: >
  What does your user need to understand about your project in order to use it.
---

Forester is a bare-metal image-based unattended provisioning service for Red Hat Anaconda (Fedora, RHEL, CentOS Stream, Alma Linux...) with simplicity of configuration and use in mind. It utilizes Redfish API and PXE BIOS/EFI or UEFI-HTTP Boot to deploy images created by Image Builder through Anaconda.

# Workflow

An example workflow for installation and operation.

## Setup

The installation workflow:

* Start the service container
* Connect to servers via out of band management and enable Redfish
* Make sure the PXE/EFI-HTTP configuration is set up on the servers
* Configure DHCP to give clients forester IP address
* Add servers into Forester inventory (credentials and Redfish URL)

## Operation

The provisioning workflow with Forester:

* Download or generate OS image with updates, user accounts, ssh keys and required configuration
* Upload the image into Forester
* Select a server from the inventory that is available
* Deploy an image to a server
* Release the server once it is ready to be reprovisioned again
* Repeat the process for new updated images

# Talks (archive)

The following talks give a brief overview of Forester. Quick introduction to Forester for Red Hat Console Q3 Hackathon 2023:

{{< youtube VK6pMjmmHhQ >}}

Demo of Forester provisioning libvirt VMs (this is only useful for development purposes):

{{< youtube jxCHU_nzluY >}}

Full [lightning talk from DevConf 2023](https://www.youtube.com/live/6nRP0si2wKI?feature=share&t=8674) (poor audio quality).
