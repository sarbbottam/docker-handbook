🐳 docker up and running
---

My notes from [docker self paced training](https://training.docker.com/self-paced-training)

---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [What is Docker?](#what-is-docker)
- [Why container?](#why-container)
  - [one application : one physical server](#one-application--one-physical-server)
  - [`virtual machine` (hypervisor based virtualization)](#virtual-machine-hypervisor-based-virtualization)
  - [container base virtualization](#container-base-virtualization)
  - [Container vs VMs](#container-vs-vms)
- [Docker concepts and terminology](#docker-concepts-and-terminology)
- [Installation](#installation)
- [Docker `hello world`](#docker-hello-world)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---
## What is Docker?

Docker is a platform for developing, shipping and running applications using container virtualization technology.

Docker platform consists of multiple products/tools
- docker engine (docker daemon or server)
- docker machine
- docker compose
- docker swarm ?
- [docker hub](https://hub.docker.com/)

## Why container?

### one application : one physical server

cons

- slow deployment
- more cost
- wasted resources
- difficult to scale and migrate
- tight coupling with hardware vendor

### `virtual machine` (hypervisor based virtualization)

pros

- better resource utilization
- easy to scale
- cloud based
- elastic - get more VM's when needed
- pay as you go

cons

- each VM still requires CPU allocation, storage, RAM, entire guest OS
- more VM's mean more resources
- guest OS implies wasted resources
- portability not guaranteed

### container base virtualization

container based virtualization uses the kernel on the host's OS to run multiple guest instances

- each guest instance is called `container`
- each container has its own
  - `root` file system
  - `process`
  - `memory`
  - `devices`
  - `network ports`

It looks and operates like a VM but it is not a VM

**container interacts with host OS' kernel to create an isolated application platform**
Within each container, application and its dependencies are installed

container isolates the run time environment

### Container vs VMs

- containers are lightweight
- no guest OS
- less CPU, RAM, storage space
- more containers per m/c than VM
- greater portability

## Docker concepts and terminology

`docker engine` is program that enables containers to be built, shipped and run.
It is also known as `docker daemon` or `docker server`.
- it uses linux kernel namespaces and control groups to create and manage `containers`.
- it uses `pid`, `net`, `ipc`, `mnt` and `uts` namespaces, to create an isolated workspace called `container`

## Installation

- For Mac/Windows, install docker application from [beta.docker.com](https://beta.docker.com/)
- For Linux, check out instructions at [docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

Verify installation by `docker version`.
If you see something like below, you are all set.
```
Client:
 Version:      1.11.0
 API version:  1.23
 Go version:   go1.5.4
 Git commit:   4dc5990
 Built:        Wed Apr 13 19:36:04 2016
 OS/Arch:      darwin/amd64

Server:
 Version:      1.11.0
 API version:  1.23
 Go version:   go1.5.4
 Git commit:   a5315b8
 Built:        Tue Apr 26 15:23:39 2016
 OS/Arch:      linux/amd64
 ```

## Docker `hello world`

```
$ docker run hello-world

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```