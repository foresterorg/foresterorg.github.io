---
title: Installation
weight: 300
description: >
  Installation process of Forester service itself.
---

## CLI

Download and extract the [CLI for your architecture](https://github.com/foresterorg/forester/releases) and try to connect to controller:

    forester-cli --url http://forester:8000 appliance list

If the service is running on localhost port 8000, you can omit the `--url` argument.

## Service

The recommended way is Podman pod.

### Podman pod

The following commands create two new volumes for postgres database and images, a new pod named `forester` with two containers named `forester-pg` and `forester-api` exposing port 8000 for the REST API.

    podman volume create forester-pg
    podman volume create forester-img
    podman volume create forester-log
    podman pod create --name forester -p 8000:8000 -p 69:6969/udp
    podman run -d --name forester-pg --pod forester \
        -e POSTGRESQL_USER=forester -e POSTGRESQL_PASSWORD=forester -e POSTGRESQL_DATABASE=forester \
        -v forester-pg:/var/lib/pgsql/data:Z quay.io/fedora/postgresql-15; sleep 5s
    podman run -d --name forester-api --pod forester \
        -e DATABASE_USER=forester -e DATABASE_PASSWORD=forester \
        -v forester-img:/images:Z -v forester-log:/logs:Z quay.io/forester/controller:latest

Communication ports:

* 8000 TCP - HTTP
* 6969 UDP - TFTP

Podman will likely not be allowed to bind port lower than 1024, unless executed as root or the lower port is allowed via:

    sudo sysctl net.ipv4.ip_unprivileged_port_start=69

In order for TFTP to work through NAT, connection tracking modules must be loaded too:

    sudo modprobe nf_conntrack_tftp
    sudo modprobe nf_nat_tftp

To make both changes permanent on Fedora or Red Hat system:

    echo "net.ipv4.ip_unprivileged_port_start=69" > /etc/sysctl.d/10-unprivileged_ports.conf
    echo "install nf_conntrack_tftp" > /etc/modprobe.d/tftp.conf
    echo "install nf_nat_tftp" >> /etc/modprobe.d/tftp.conf

To uninstall everything up including data and images:

    podman rm -f forester-pg forester-app
    podman pod rm -f forester
    podman volume rm forester-img forester-pg

### Docker compose

To start the Forester via docker-compose or podman-compose:

    curl https://raw.githubusercontent.com/foresterorg/forester/main/compose.yaml > compose.yaml
    docker-compose up -d

### Compiling from source

Compilation is described in the [Contributing](/docs/contributing) chapter.
