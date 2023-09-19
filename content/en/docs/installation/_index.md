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
    podman pod create --name forester -p 8000:8000
    podman run -d --name forester-pg --pod forester -e POSTGRESQL_USER=forester -e POSTGRESQL_PASSWORD=forester -e POSTGRESQL_DATABASE=forester -v forester-pg:/var/lib/pgsql/data:Z quay.io/fedora/postgresql-15; sleep 5s
    podman run -d --name forester-api --pod forester -e DATABASE_USER=forester -e DATABASE_PASSWORD=forester -e IMAGES_DIR=/img -v forester-img:/img:Z quay.io/forester/controller:latest

To uninstall everything up including data and images:

    podman rm -f forester-pg forester-app
    podman pod rm -f forester
    podman volume rm forester-img forester-pg

### Docker compose

To start the Forester controller and Postgres database on port 8000 with data for both database and images stored in `./data` directory run:

    curl https://raw.githubusercontent.com/foresterorg/forester/main/compose.yaml > compose.yaml
    EXPOSED_APP_PORT=8000 DATA_DIR=./data podman-compose up -d

This command requires [podman-compose](https://github.com/containers/podman-compose) (available in Fedora and EPEL). You can use docker-compose as well, it is known to work but be aware we do not test against Docker during development.

To remove Forester completely, stop containers and remove them, and the data directory.

### Compiling from source

Compilation is described in the [Contributing](/docs/contributing) chapter.
