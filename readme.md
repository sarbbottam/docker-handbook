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