---
title: Contributing
weight: 900
description: >
  How to contribute to the Forester project.
---

## Code changes

Both service and CLI are written in Go, we format the code with `go fmt` and stick with the widely adopted Go best practices.

## Compiling from sources

Build the project, the script will also install required CLI tools for code generation and database migration:

    git clone https://github.com/foresterorg/forester
    cd forester
    ./build.sh

When you start the backend for the first time, it will migrate database (create tables). By default, it connect to "localhost" database "forester" and user "postgres".

    ./forester-controller

## Libvirt setup

Configure libvirt environment for booting from network via UEFI HTTP Boot, add the five "dnsmasq" options into the "default" libvirt network. Also, optionally configure PXEv4 and IPEv6 to return a non-existing file ("USE_HTTP" in the example) to speed up OVMF firmware to fallback to HTTPv4:

Edit the default network

    sudo virsh net-edit default

To the following (keep your uuid and IP configuration)

    <network xmlns:dnsmasq='http://libvirt.org/schemas/network/dnsmasq/1.0'>
      <name>default</name>
      <uuid>9f3e4377-3d33-42df-b34c-7880295d24ee</uuid>
      <forward mode='nat'/>
      <bridge name='virbr0' zone='trusted' stp='on' delay='0'/>
      <mac address='52:54:00:7a:00:01'/>
      <ip address='192.168.122.1' netmask='255.255.255.0'>
        <dhcp>
          <range start='192.168.122.2' end='192.168.122.254'/>
          <bootp file='USE_HTTP'/>
        </dhcp>
      </ip>
      <dnsmasq:options>
        <dnsmasq:option value='dhcp-vendorclass=set:bios,PXEClient:Arch:00000'/>
        <dnsmasq:option value='dhcp-vendorclass=set:efi,PXEClient:Arch:00007'/>
        <dnsmasq:option value='dhcp-vendorclass=set:efix64,PXEClient:Arch:00009'/>
        <dnsmasq:option value='dhcp-vendorclass=set:efihttp,HTTPClient:Arch:00016'/>
        <dnsmasq:option value='dhcp-option-force=tag:efihttp,60,HTTPClient'/>
        <dnsmasq:option value='dhcp-match=set:ipxe-http,175,19'/>
        <dnsmasq:option value='dhcp-match=set:ipxe-https,175,20'/>
        <dnsmasq:option value='dhcp-match=set:ipxe-menu,175,39'/>
        <dnsmasq:option value='dhcp-match=set:ipxe-pxe,175,33'/>
        <dnsmasq:option value='dhcp-match=set:ipxe-bzimage,175,24'/>
        <dnsmasq:option value='dhcp-match=set:ipxe-iscsi,175,17'/>
        <dnsmasq:option value='dhcp-match=set:ipxe-efi,175,36'/>
        <dnsmasq:option value='tag-if=set:ipxe-ok,tag:ipxe-http,tag:ipxe-menu,tag:ipxe-iscsi,tag:ipxe-pxe,tag:ipxe-bzimage'/>
        <dnsmasq:option value='tag-if=set:ipxe-ok,tag:ipxe-http,tag:ipxe-menu,tag:ipxe-iscsi,tag:ipxe-efi'/>
        <dnsmasq:option value='dhcp-boot=tag:bios,bootstrap/ipxe/undionly.kpxe,,192.168.122.1'/>
        <dnsmasq:option value='dhcp-boot=tag:!ipxe-ok,tag:efi,bootstrap/ipxe/ipxe-snponly-x86_64.efi,,192.168.122.1'/>
        <dnsmasq:option value='dhcp-boot=tag:!ipxe-ok,tag:efi64,bootstrap/ipxe/ipxe-snponly-x86_64.efi,,192.168.122.1'/>
        <dnsmasq:option value='dhcp-boot=tag:!ipxe-ok,tag:efihttp,http://192.168.122.1:8000/bootstrap/ipxe/ipxe-snponly-x86_64.efi'/>
        <dnsmasq:option value='dhcp-boot=tag:ipxe-ok,tag:!efihttp,bootstrap/ipxe/chain.ipxe,,192.168.122.1'/>
        <dnsmasq:option value='dhcp-boot=tag:ipxe-ok,tag:efihttp,http://192.168.122.1:8000/bootstrap/ipxe/chain.ipxe'/>
      </dnsmasq:options>
    </network>

Make sure to update the HTTP address in case you want to use different network than "defalut" (which is 192.168.122.0). Restart the network to make the DHCP server settings effective:

    sudo virsh net-destroy default
    sudo virsh net-start default

When testing, create three VMs:

* BIOS for PXE testing
* EFI for PXE EFI testing, SecureBoot must be turned off in the firmware
* EFI-HTTP for HTTP EFI testing, SecureBoot must be turned off in the firmware

Note you cannot use libvirt for real provisioning as there are some issues with libvirt driver, it does not boot into the OS after restart: https://github.com/foresterorg/forester/issues/6

## Development of TFTP

The built-in TFTP server is running on port 6969 by default to allow development on non-root accounts. To start and debug the application's built in TFTP listener as a regular (developer) account, simply forward the traffic:

    sudo iptables -t nat -I PREROUTING -p udp --dport 69 -j REDIRECT --to-ports 6969

When using firewalld

    sudo firewall-cmd --add-forward-port=port=69:proto=udp:toport=6969

Make sure to add `--zone=libvirt` when testing Forester via libvirt and `--permanent` to set it permanently

Note this will not work when running the application via podman or docker, you can only use this for local development.

## Redfish emulators

There are several emulators available.

The official one unfortunatelly does not allow updates (reboot), you can still try it out:

    podman run --rm -p 5000:5000 dmtf/redfish-interface-emulator:latest

However, there is another one from DMTF, which does support updates:

    podman run --rm -p 5000:8000 dmtf/redfish-mockup-server:latest

## Enroll X509 into EFI

- Sign server.cer with ca.cer
- Configure the service to use it (TBD)
- mkfs.msdos -C letsencrypt.img 300
- mcopy -i letsencrypt.img letsencrypt.cer ::/ca.cer
- mdir -i letsencrypt.img
- Enroll the certificate into EFI

