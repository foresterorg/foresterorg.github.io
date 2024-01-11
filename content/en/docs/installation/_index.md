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
    podman pod create --name forester -p 8000:8000 -p 6969:6969/udp
    podman run -d --name forester-pg --pod forester -e POSTGRESQL_USER=forester -e POSTGRESQL_PASSWORD=forester -e POSTGRESQL_DATABASE=forester -v forester-pg:/var/lib/pgsql/data:Z quay.io/fedora/postgresql-15; sleep 5s
    podman run -d --name forester-api --pod forester -e DATABASE_USER=forester -e DATABASE_PASSWORD=forester -e IMAGES_DIR=/img -v forester-img:/img:Z quay.io/forester/controller:latest

Communication ports:

* 8000 - HTTP
* 6969 - TFTP

Since exporting port 69 for the TFTP service would require root, we recommend to forward port 69 instead. This is only needed when using PXE, in case of UEFI-HTTP booting the TFTP service is unused:

    sudo iptables -t nat -I PREROUTING -p udp --dport 69 -j REDIRECT --to-ports 6969

Or when using firewalld:

    sudo firewall-cmd --add-forward-port=port=69:proto=udp:toport=6969

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
