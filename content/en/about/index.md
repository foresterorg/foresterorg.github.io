---
title: Forester Setup
linkTitle: Setup
menu: {main: {weight: 10}}
---

{{% blocks/cover title="Forester Setup" image_anchor="bottom" height="auto" %}}

A brief overview of Forester installation and example commands.
Visit our [documentation](/docs/) for more detailed information.
{.mt-5}

{{% /blocks/cover %}}

{{% blocks/lead %}}

Forester has been designed with simplicity in mind, you only need an Anaconda OS image which you can generate on [Red Hat Portal](https://console.redhat.com/insights/image-builder) for free, and access to bare-metal hardware via out-of-band management with Redfish (e.g. iDRAC). Scroll down for installation and setup instructions.

{{% /blocks/lead %}}

{{% blocks/section %}}

# Service installation

The Forester service provides all functionality through port 8000 (HTTP) and stores some data in Postgres database. The recommended installation is via podman:

    podman volume create forester-pg
    podman volume create forester-img
    podman pod create --name forester -p 8000:8000
    podman run -d --name forester-pg --pod forester \
        -e POSTGRESQL_USER=forester \
        -e POSTGRESQL_PASSWORD=forester \
        -e POSTGRESQL_DATABASE=forester \
        -v forester-pg:/var/lib/pgsql/data:Z \
        quay.io/fedora/postgresql-15; sleep 5s
    podman run -d --name forester-api --pod forester \
        -e DATABASE_USER=forester \
        -e DATABASE_PASSWORD=forester \
        -e IMAGES_DIR=/img \
        -v forester-img:/img:Z \
        quay.io/forester/controller:latest

Hardware requirements match Postgres, as long as there is enough CPU and memory for Postgres, Forester will run just fine. 

Change `-p` argument if you wish to use different port than 8000. Uninstallation is easy and is covered in the [documentation](/docs/).

Download and extract the [CLI for Linux/Mac/Windows](https://github.com/foresterorg/forester/releases) named `forester-cli`.

{{% /blocks/section %}}

{{% blocks/section %}}

# Managed hardware setup

Enable Redfish protocol in server hardware, an example for Dell iDRAC:

![Dell iDRAC: Redfish](/img/idrac_redfish.png)

Enable HTTP Boot network booting and enter HTTP URI to `http:/forester:8000/boot/shim.efi`, an example for Dell iDRAC:

![Dell iDRAC: Redfish](/img/idrac_httpboot.png)

Alternatively, URI can be provided by a DHCP server. Read more in the [documentation](/docs/).

A user account should be created with the following permissions:

* Read/list information about system(s).
* Power operations (soft and hard power cycles).

For POCs, root account can be used.

{{% /blocks/section %}}

{{% blocks/section %}}

# Prepare environment

Register Redfish appliance by providing an URI to Redfish API of all systems which are supposed to be managed by Forester:

    forester-cli appliance create --kind redfish --name dellr350-a14 --uri https://root:calvin@dr350-a14.local

Every recognized system has one or more MAC addresses and an auto-generated name.

    forester-cli system list
    ID  Name        Hw Addresses           Acquired  Facts
    1   Lynn Viers  6c:fe:54:70:60:10 (4)  false     Dell Inc. PowerEdge R350

Upload an image generated from [Image Builder](https://console.redhat.com/insights/image-builder) or your own [osbuild instance](https://www.osbuild.org/guides/introduction.html). Alternatively, there is an on-prem web interface named `cockpit-composer` for [Cockpit](https://cockpit-project.org), or a CLI. Note, the official Fedora or Red Hat Enterprise Linux installer images will not work, you have to generate your own image:

    forester-cli image upload --name RHEL9-2023-11a rhel9-image-from-portal.iso

# Deploy images

To install an image onto a system, perform `acquire` operation and provide both image name and system name or one of its MAC addresses:

    forester-cli system acquire Lynn --imagename RHEL9-2023-11a

To release a system and put it back to the pool of available systems:

    forester-cli system release Lynn

{{% /blocks/section %}}

