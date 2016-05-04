🐳 Docker Build and Push Image
---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Building Docker Images](#building-docker-images)
- [New image by tagging](#new-image-by-tagging)
- [Docker Image from a Docker Container](#docker-image-from-a-docker-container)
- [Build image from `Dockerfile`](#build-image-from-dockerfile)
- [Dockerfile](#dockerfile)
- [Dockerfile instructions](#dockerfile-instructions)
- [`RUN` instruction](#run-instruction)
- [`CMD` instruction](#cmd-instruction)
- [`ENTRYPOINT` instruction](#entrypoint-instruction)
- [Pushing images to Docker Hub](#pushing-images-to-docker-hub)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---
## Building Docker Images

New `image` can be created by

- tagging and exiting `image` with a new name
  - it just a name (tag) change, the functionality remains the same as the `old image`
- making changes in a `container` and `commiting` the changes
- building from a `Dockerfile`

## New image by tagging

```
$ docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
```

## Docker Image from a Docker Container

- `docker commit` saves changes in a `container` as a new `image`
- `$ docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`
  - if no `[TAG]` is specified, `docker` will use `latest`
  - if no `REPOSITORY` is specified `REPOSITORY` and `TAG` will be `none`
- let's see it in action
  - `$ docker run -it ubuntu bash`
  - in the `ubuntu` `container` `$ apt-get install -y curl`
    - if you find `E: Unable to locate package curl` error
      - `$ apt-get -qq update`
      - `$ apt-get -qq -y install curl`
      - refer [this StackoOverflow post](http://stackoverflow.com/questions/27273412/cannot-install-packages-inside-docker-ubuntu-image) for further details
  - if the `prompt` seems to be hanging, try pressing `return` or `ctrl + c` and retry
  - `exit` from the `container`
  - get the `container-id`, by listing the `container`
  ```sh
  $ docker ps -a
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
  9442903e27cc        ubuntu              "bash"              8 minutes ago       Exited (0) 3 seconds ago                       sad_lichterman
  ```
  - `$ docker commit 9442903e27cc curl`
  - list `docker images`
  ```sh
  $ docker images
  REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
  curl                      latest              2534caf55869        8 seconds ago       173.5 MB
  ubuntu                    latest              44776f55294a        4 days ago          120.1 MB
  ```
  - run the `image`, with `docker run`
  ```sh
  $ docker run -it curl bash
  root@86d61ffb3dac:/# which curl
  /usr/bin/curl
  root@86d61ffb3dac:/#
  ```

## Build image from `Dockerfile`

- `$ docker build [OPTIONS] PATH | URL | -`
- `PATH` is the build `context`
- `docker build` will look for `Dockerfile` at the root of the `context`
  - use `-f <file-name>` to specify a custom docker file
- option `-t [repository:tag]` will tag the `image`

Consider the following `Dockerfile`

```Dockerfile
FROM ubuntu
RUN apt-get -qq update
RUN apt-get -qq -y install curl
```

Let's create an `image` from the above `Dockerfile`
```sh
$ docker build -t curl .
Sending build context to Docker daemon 261.6 kB
Step 1 : FROM ubuntu
 ---> 44776f55294a
Step 2 : RUN apt-get -qq update
 ---> Running in ba9c3738f17b

 ---> fdbb7e8242d7
Removing intermediate container ba9c3738f17b
Step 3 : RUN apt-get -qq -y install curl
 ---> Running in e03c750ff71a

 ---> 33374acd807e
Removing intermediate container e03c750ff71a
Successfully built 33374acd807e
```
- verify the `image` built, via `docker images`
```sh
$ docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
curl                      latest              33374acd807e        2 minutes ago       173.5 MB
```

## Dockerfile

> A `Dockerfile` is a configuration file that contains instructions for building a `Docker image`

- more effective than `docker commit`
- create `image` multiple times via `docker build` without carrying out the redundant steps manually
- easily fits in CI/CD process

## Dockerfile instructions

Instructions specify what to do while building `image`
- `FROM` specifies `base image`
  - every `image` needs a `base image`
- `RUN` specifies which command to execute
```Dockerfile
FROM ubuntu
RUN apt-get -qq update
RUN apt-get -qq -y install curl
```

## `RUN` instruction

- each `RUN` instruction will execute the `command` on top of `writable layer` and `commit` the changes
- aggregate multiple `RUN` instructions using `&&` to avoid multiple `commit`
```Dockerfile
RUN apt-get -qq update && apt-get -qq -y install curl
```

## `CMD` instruction

- `CMD` defines the `default command`, that will be executed when the `container` is executed
- `CMD` has no role during `image` creation
- **only one** `CMD`, the last one wins
- can be overridden at run time
- `CMD` can be in  three forms
  - `CMD ["executable","param1","param2"]` (exec form, this is the preferred form)
  - `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT)
  - `CMD command param1 param2` (shell form)

Let's create a new image with `CMD`

```Dockerfile
FROM ubuntu

RUN apt-get -qq update
RUN apt-get -qq -y install curl
RUN apt-get -qq -y install iputils-ping

CMD ["ping", "127.0.0.1",  "-c", "30"]
```
- `$ docker build -t ping .` to build a new image
- run the new `image` via `docker run`
```sh
$ docker run ping
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.039 ms
... 30 times
```
- passing a command via `docker run` will override the default `CMD`
```sh
$ docker run ping echo hello world
hello world
```

## `ENTRYPOINT` instruction

An `ENTRYPOINT` allows you to configure a `container` that will run as an executable.
For example, the following will start nginx with its default content, listening on port 80.

```sh
docker run -i -t --rm -p 80:80 nginx
```

- `ENTRYPOINT` can be in  two forms
  - `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred)
  - `ENTRYPOINT command param1 param2 (shell form)`
- `ENTRYPOINT` in `exec` form will accept arguments passed via `docker run`
- create an image with `ENTRYPOINT`

```Dockerfile
FROM ubuntu

RUN apt-get -qq update
RUN apt-get -qq -y install curl
RUN apt-get -qq -y install iputils-ping

ENTRYPOINT ["ping"]
```

- `$ docker build -t ping .` to build a new `image`
- run the new `image` via `docker run` without any arguments

```sh
$ docker run ping
Usage: ping [-aAbBdDfhLnOqrRUvV] [-c count] [-i interval] [-I interface]
            [-m mark] [-M pmtudisc_option] [-l preload] [-p pattern] [-Q tos]
            [-s packetsize] [-S sndbuf] [-t ttl] [-T timestamp_option]
            [-w deadline] [-W timeout] [hop1 ...] destination
```

- run the new `image` via `docker run` with arguments

```sh
$ docker run ping 127.0.0.1 -c 5
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.043 ms
...
5 packets transmitted, 5 received, 0% packet loss, time 3996ms
rtt min/avg/max/mdev = 0.037/0.040/0.043/0.008 ms
```

## Pushing images to Docker Hub

- create an account, if you dont have one at [hub.docker.com](https://hub.docker.com)
- navigate to [add repo](https://hub.docker.com/add/repository/)
  - and follow the instructions
- once the repo is created at [hub.docker.com](https://hub.docker.com), use `$ docker push repo:tag` to push it to docker hub.
- prior pushing authenticate docker via hub.docker.com credentials
  - `$ docker login` and follow the promt and instrunctions
- `$ docker push sarbbottam/hello-world`
