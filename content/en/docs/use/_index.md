--- 
title: Use
weight: 300
description: >
  Operation instructions.
---

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

To create appliance for hacking and development a good appliance type is libvirt through local UNIX socket:

    forester-cli appliance create --kind libvirt --name local

Or via TCP connection (TLS is not supported):

    forester-cli appliance create --kind libvirt --name remote --uri tcp://my.libvirt.local:16509

Warning: username and password are currently stored as clear text and fully readable through the API.

    forester-cli appliance list
    ID  Name     Kind  URI
    1   libvirt  1     unix:///var/run/libvirt/libvirt-sock
    2   dellr350 2     https://root:calvin@dr350-a14.local

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
Appliance URI   https://root:calvin@dell-r350-08-drac.mgmt.sat.rdu2.redhat.com
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

## Deploy images

Systems are either **released** or **acquired**. By acquisition, an operator performs installation of a specific image onto the hardware:

    forester-cli system acquire lynn --imagename RHEL9

To release a system and put it back to the pool of available systems:

    forester-cli system release lynn

Warning: There is no authentication or authorization in the API, anyone can acquire or release systems or even add new appliances.
