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

    sudo virsh net-edit default
    <network xmlns:dnsmasq='http://libvirt.org/schemas/network/dnsmasq/1.0'>
      <name>default</name>
      <uuid>9f3e4377-3d33-42df-b34c-7880295d24ee</uuid>
      <forward mode='nat'/>
      <bridge name='virbr0' zone='trusted' stp='on' delay='0'/>
      <mac address='52:54:00:7a:00:01'/>
      <ip address='192.168.122.1' netmask='255.255.255.0'>
        <tftp root='/var/lib/tftpboot'/>
        <dhcp>
          <range start='192.168.122.2' end='192.168.122.254'/>
          <bootp file='USE_HTTP'/>
        </dhcp>
      </ip>
      <ip family='ipv6' address='2001:db8:dead:beef:fe::2' prefix='96'>
        <dhcp>
          <range start='2001:db8:dead:beef:fe::1000' end='2001:db8:dead:beef:fe::2000'/>
        </dhcp>
      </ip>
      <dnsmasq:options>
        <dnsmasq:option value='dhcp-vendorclass=set:efi-http,HTTPClient:Arch:00016'/>
        <dnsmasq:option value='dhcp-option-force=tag:efi-http,60,HTTPClient'/>
        <dnsmasq:option value='dhcp-boot=tag:efi-http,&quot;http://192.168.122.1:8000/boot/shim.efi&quot;'/>
      </dnsmasq:options>
    </network>

Make sure to update the HTTP address in case you want to use different network than "defalut" (which is 192.168.122.0). Restart the network to make the DHCP server settings effective:

    sudo virsh net-destroy default
    sudo virsh net-start default

Now, boot an empty UEFI VM on this network from network, it must be set for UEFI HTTP Boot. Enter EFI firmware by pressing ESC key after it is powered on, enter "Boot manager" screen and make "UEFI HTTPv4 [MAC]" the first boot option.

Note you cannot use libvirt for real provisioning as there are some issues with libvirt driver, it does not boot into the OS after restart: https://github.com/foresterorg/forester/issues/6

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

