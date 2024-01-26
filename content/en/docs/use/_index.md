--- 
title: Use
weight: 300
description: >
  Operation instructions.
---

## Build an image

Images can be built on [Red Hat Console](https://console.redhat.com/insights/image-builder) or locally. Follow the instructions to perform build on your own hardware.

Install composer and composer CLI and start the build service:

    sudo dnf install osbuild-composer composer-cli
    sudo systemctl enable --now osbuild-composer.socket

For more information on how to setup permissions for non-root accounts, [read the project documentation](https://www.osbuild.org/guides/image-builder-on-premises/installation.html).

Create a blueprint file named `base-image-with-vim.toml` with the following contents:

    name = "base-image-with-vim"
    description = "A base system with vim"
    version = "0.0.1"
    [[packages]]
    name = "vim"
    version = "*"

Read the osbuild documentation for more information on customization of the image. Now, upload the blueprint file into the service:

    composer-cli blueprints push base-image-with-vim.toml

To build a Fedora/RHEL installation image:

    composer-cli compose start base-image-with-vim image-installer

Wait until it is done and then download the result tarball:

    composer-cli compose results UUID

You may proceed to the next chapter. To build a Fedora IoT (or RHEL for Edge) installation image, create a blueprint file named `empty.toml` with the following contents:

    cat empty.toml
    name = "empty"
    description = "Empty blueprint"
    version = "0.0.1"

Push the empty blueprint:

    composer-cli blueprints push empty.toml

Build and download a new ostree repository:

    composer-cli compose start base-image-with-vim iot-commit
    composer-cli compose results UUID

The ostree repository needs to be published over HTTP protocol, this is not part of this turorial.

    composer-cli compose start-ostree --ref "fedora/38/x86_64/iot" --url http://zzzap.tpb.lab.eng.brq.redhat.com/~lzap/f38-iot-commit/repo/ empty iot-installer
    composer-cli compose results UUID

When building RHEL instead of Fedora, replace `iot-installer` with `edge-installer`. Also the `ref` will be different, typically `rhel/8/x86_64/edge`.

Use the following command to watch the progress of the build:

    composer-cli compose info UUID

Once the build is finished, download the ISO file with:

    composer-cli compose results UUID

Composes can be deleted to save disk space using `delete` command.

## Upload an image

The first step is to upload the image:

    forester-cli image upload --name Fedora37 f37-minimal-image.iso

Check it:

    forester-cli image list
    Image ID  Image Name
    1         RHEL9
    2         Fedora37

    forester-cli image show Fedora37
    Attribute  Value
    ID         2
    Name       Fedora37

## Configure Redfish

To create a Redfish appliance use kind named "redfish":

    forester-cli appliance create --kind redfish --name dellr350 --uri https://root:calvin@dr350-a14.local

To create appliance for hacking and development a good appliance type is libvirt through local UNIX socket, an example for system session:

    forester-cli appliance create --kind libvirt --name system --uri qemu:///system

An example for user session (URI accepts both `unix` socket or `qemu` paths):

    sudo usermod -a -G libvirt $(whoami)
    forester-cli appliance create --kind libvirt --name session --uri qemu:///session

Or via TCP connection (TLS is not supported):

    forester-cli appliance create --kind libvirt --name remote --uri tcp://host.containers.internal:16509

Replace `host.containers.internal` with the Forester hostname, this is a special name that will work for Podman to access host system. To access libvirt over TCP, it must be configured to do so:

    grep '^[^#]' /etc/libvirt/libvirtd.conf
    auth_tcp = "none"

And:

    systemctl enable --now libvirtd-tcp.socket

It is possible to create no operation appliance which does nothing or `redfish_manual` appliance which performs detection of systems, however, it does not perform any power operations. With `noop` appliance, systems need to be registered manually whereas with `redfish_manual` systems can be detected automatically.

Warning: username and password are currently stored as clear text and fully readable through the API.

    forester-cli appliance list
    ID  Name     Kind  URI
    1   system   1     unix:///var/run/libvirt/libvirt-sock
    2   session  1     qemu:///session
    3   dellr350 2     https://root:calvin@dr350-a14.local

## Discover systems

Discover the system or multiple blades in chassis:

    forester-cli appliance enlist dellr350

One or more systems are available now, each system has an unique ID, one or more MAC addresses and randomly generated name. A system can be referenced via both MAC address and random name:

```
forester-cli system list
ID  Name        Hw Addresses           Acquired  Facts
1   Lynn Viers  6c:fe:54:70:60:10 (4)  false     Dell Inc. PowerEdge R350
```

To show more details of a system:

```
forester-cli system show Viers
Attribute       Value
ID              1
Name            Lynn Viers
Acquired        false
Acquired at     Mon Sep  4 14:40:50 2023
Image ID        1
MAC             6c:fe:54:70:60:10
MAC             c4:5a:b1:a0:f2:b5
MAC             6c:fe:54:70:60:11
MAC             c4:5a:b1:a0:f2:b4
Appliance Name  dell
Appliance Kind  2
Appliance URI   https://root:calvin@dr350-a14.local
UID             4c4c4544-004c-3510-804c-c4c04f435731

Fact                     Value
baseboard-asset-tag      
baseboard-manufacturer   Dell Inc.
baseboard-product-name   0MTYYT
baseboard-serial-number  .DL5XXXX.MXWSG0000000HE.
baseboard-version        A02
bios-release-date        11/14/2022
bios-revision            1.5
bios-vendor              Dell Inc.
bios-version             1.5.1
chassis-asset-tag        Not Specified
chassis-manufacturer     Dell Inc.
chassis-serial-number    DL5XXXX
chassis-type             Rack Mount Chassis
chassis-version          Not Specified
cpuinfo-processor-count  4
firmware-revision        
memory-bytes             8201367552
processor-family         Xeon
processor-frequency      2800 MHz
processor-manufacturer   Intel
processor-version        Intel(R) Xeon(R) E-2314 CPU @ 2.80GHz
redfish_asset_tag        
redfish_description      Computer System which represents a machine (physical or virtual) and the local resources such as memory, cpu and other devices that can be accessed from that machine.
redfish_manufacturer     Dell Inc.
redfish_memory_bytes     8589934592
redfish_model            PowerEdge R350
redfish_name             System
redfish_oid              /redfish/v1/Systems/System.Embedded.1
redfish_part_number      0MTYYTA02
redfish_pcie_dev_count   9
redfish_processor_cores  4
redfish_processor_count  1
redfish_processor_model  Intel(R) Xeon(R) E-2314 CPU @ 2.80GHz
redfish_serial_number    MXWSJ0032100HI
redfish_sku              DL5XXXX
serial                   DL5XXXX
system-family            PowerEdge
system-manufacturer      Dell Inc.
system-product-name      PowerEdge R350
system-serial-number     DL5XXXX
system-sku-number        SKU=0A94;ModelName=PowerEdge R350
system-uuid              4c4c4544-004c-3510-804c-c4c04f435731
system-version           Not Specified
```

Facts which start with `redfish` were recognized via Redfish API, other facts can be discovered by booting the system into Anaconda in a released state:

    forester-cli appliance bootnet lynn

## Enter system manually

An alternative way of registering systems is to enter them manually, this is useful for `noop` appliance:

    forester-cli system register --name my-system-13 --hwaddrs AA:BB:CC:DD:EE:FF --facts test=1 --appliance noop --uid unique_uuid

Every system must have an appliance assigned, a `noop` (no operation) appliance can be used and unique string (typically UUID or random number).

## Customizing installation

Installation can be further customized using [Anaconda Kickstart syntax](https://pykickstart.readthedocs.io/en/latest/kickstart-docs.html) via snippets. There are several kinds of snippets which can be optionally associated with a system:

* disk (partitioning layout snippet)
* rootpw (root password snippet)
* security (security-related snippet)
* locale (locale-related snippet)
* post (`%post` installation snippet)

To list all snippets:

    forester-cli snippet list
    ID  Name           Kind
    1   SingleVolume   disk
    2   SharedPass     rootpw
    3   InstallAnsible post

To create a snippet:

    forester-cli snippet create --name MySnippet --kind locale

An editor (depending on the `$EDITOR` environment variable) is launched, when saved the snippet is uploaded to the service. An example content:

    lang cs_CZ.UTF-8
    keyboard 'cs'
    timezone Europe/Prague --utc

To edit or delete snippet, use appropriate `edit` or `delete` CLI subcommands.

A system can have zero to any number of snippets associated and one extra "custom snippet" which can be used to provide one or more ad-hoc Kickstart lines without storing anything into the database.

## Deploy images

Deploying is done via `deploy` command:

    forester-cli system deploy lynn --imagename RHEL9

To customize installation with snippets:

    forester-cli system deploy lynn --imagename RHEL9 --snippets SingleVolume SharedPass --customsnippet "%pre\necho Hello\n%end\n"

Warning: There is no authentication or authorization in the API, anyone can deploy or even add new appliances.

## Troubleshooting

Both hardware discovery and installation are handled by Anaconda which takes Kickstart as the input. To view current Kickstart of an existing system do:

    forester-cli system kickstart lynn

Depending on the state of the system, it will be either discovery Kickstart, or installation kickstart. Kickstart can be validated using `ksvalidator` command for syntax errors.

Anaconda installer is configured to send all logs via syslog protocol into Forester. To list all available logs of a system:

    forester-cli system logs lynn

To view log contents of an installation:

    forester-cli system logs lynn -d f-1-06fe9588-b50e-4ee7-8e81-1f87a8-b265e1.log

Logs can be viewed as they arrive in the service, there is no "watch" feature available tho.

