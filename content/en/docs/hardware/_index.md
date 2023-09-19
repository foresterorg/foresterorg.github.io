---
title: Hardware setup
weight: 200
description: >
  Actions which need to be done on hardware which is subject of provisioning.
---

Servers need to boot via UEFI HTTP Boot (not UEFI PXE) a particular URL `http://forester:8000/boot/shim.efi` (where `forester` is a machine running the Forester controller container). There are currently two options how to achieve that.

## Static URL

First option, which is great for PoC or testing out Forester on just a handful of machines, is to configure the HTTP UEFI Boot URL in BIOS directly. For example, on Dell iDRAC go to Configuration - BIOS Settings - Network Settings and enable HTTP device and set the URI. Then apply and commit the change (requires reboot). To speed the boot process up, disable PXE devices on the same page.

## DHCP

Second option is to configure the URL on the DHCP server so no changes are required through out of band management. Example configuration for ISC DHCPv4 (also works on DHCPv6):

```
class "httpclients" {
  match if substring (option vendor-class-identifier, 0, 10) = "HTTPClient";
  option vendor-class-identifier "HTTPClient";
  filename "http://forester:8000/boot/shim.efi";
}
subnet 192.168.42.0 netmask 255.255.255.0 {
  range dynamic-bootp 192.168.42.100 192.168.42.120;
  default-lease-time 14400;
  max-lease-time 172800;
}
```
