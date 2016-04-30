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
- [Docker client and daemon](#docker-client-and-daemon)
- [Docker containers and images](#docker-containers-and-images)
- [Docker orchestration](#docker-orchestration)
- [Benefits of Docker](#benefits-of-docker)
- [Docker images](#docker-images)
- [Docker containers](#docker-containers)
- [Docker container with terminal](#docker-container-with-terminal)
- [More on containers ...](#more-on-containers-)
- [Containers in detached mode](#containers-in-detached-mode)
- [Image layers](#image-layers)
- [Container writable layer](#container-writable-layer)

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

## Docker client and daemon

- `docker` follows `client/server` architecture
- `client` passes the user inputs to `daemon` or `server`
- `daemon` builds, runs and manages `containers`
- `client` and `server` are most of the time in same host, but can be in different hosts too
- `$ docker version` will provide details of `client` as well as `server`

## Docker containers and images

images
- read only templates used to create containers

containers
- isolated application platform
- contains everything to run an application, packaged with dependencies
- based on one or more images

## Docker orchestration

- `docker-compose` - create and manage multi-container application
- `docker-swarm` - clusters many engine and schedule containers

## Benefits of Docker

- separation of concerns
  - developers focus on building the apps
  - sys admins focus on the deployement
- faster development cycle
- application portability
- scalability
- more apps on one host machine

## Docker images

- `$ docker images` will list the local `images`
```sh
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              94df4f0ce8a4        3 days ago          967 B
```
- `$ docker run <image>` will fetch the image from the `docker registry` (most likely hub.docker.com) if not available locally
- `images` are specified by `repository:tag`
- if no tag is specified `docker` will use the tag `latest`

## Docker containers

- `$ docker run [options] image [command] [arg...]` will `create` and `run` a `container` from the specified `image`
- `image` is specified with `repository:tag`
- `$ docker run ubuntu ps -ef`
```sh
# will download the image if not available locally ...
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 03:09 ?        00:00:00 ps -ef
```

## Docker container with terminal

- use `-i` & `-t` `options` with `docker run`
  - `-i` to connect to STDIN on container
  - `-t` to get pseudo-terminal
```sh
$ docker run -i -t ubuntu bash
root@0538835f124a:/#
```
  - `0538835f124a` is the `container id`

## More on containers ...
- `$ docker run image` **will always create a new** `container`
- use `$ docker start container-id` **to start an existing container**
- to list existing container use `$ docker ps -a`
- use `$ docker ps` to list only the active containers
```sh
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                          PORTS               NAMES
0538835f124a        ubuntu              "bash"              8 minutes ago       Exited (0) About a minute ago                       mad_mirzakhani
43e171ebaa97        ubuntu              "ps -ef"            14 minutes ago      Exited (0) 14 minutes ago                           elated_wing
913597689e93        hello-world         "/hello"            5 hours ago         Exited (0) 5 hours ago                              suspicious_lovelace
```
- a container runs as long as the process corresponding to the `command`, specified in the `docker run command` runs
- command's `pid` is always 1
- `$ docker run ubuntu ps -ef`
```sh
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 03:09 ?        00:00:00 ps -ef
```
- docker provides a random name to the container if not explicitly provided

## Containers in detached mode

- `$ docker run -d image command`; `-d` tells `docker` to run the container in detached mode, in background
  - make sure the `command` provided runs a hanging process, like run a web server, otherwise the container will stop as soon as the process with `PID 1` exits
- `$ docker logs container-id` to inspect the output of the container
  - `$ docker logs -f container-id` is similat to `tail -f`
- `$ docker run -d -P tomcat` will run tomcat in detached mode
  - `-P` tell `docker` to map `container port` to `host port`
```sh
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
3c4e535cfae8        tomcat              "catalina.sh run"   3 seconds ago       Up 2 seconds        0.0.0.0:32769->8080/tcp   hungry_pike
```
- checkout `IP:PORT` in browser; for example `0.0.0.0:32769` w.r.t. above `docker ps`
- if no command is specified to `docker run` it would execute the `default` command.
  - for tomcat its `catalina.sh run`

## Image layers

- `images` are comprised of multiple layers
- a **`layer` is just another image**
- every `image` contains a base layer
- layers are read only

## Container writable layer

- when a container is launched from an image, docker adds a writable layer on top.
  - a read/write file system on top of all other layers
- the process that the container will run will be run in this read/write layer
- any changes made will be made to this read/write layer
- all the other layers are readonly
- `docker` uses `copy on write system`
  - a file from read only layer in copied  to a writable layer
  - original read only version is hidden
  - changes happen in writable layer
  - `copy on write system` makes the spinning up container fast