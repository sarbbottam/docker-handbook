🐳 Docker fundamentals
---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Docker images](#docker-images)
- [Docker containers](#docker-containers)
- [Docker container with terminal](#docker-container-with-terminal)
- [More on containers ...](#more-on-containers-)
- [Containers in detached mode](#containers-in-detached-mode)
- [Image layers](#image-layers)
- [Container writable layer](#container-writable-layer)
- [`docker commit`](#docker-commit)
- [Dockerfile](#dockerfile)
  - [What is Dockerfile?](#what-is-dockerfile)
  - [Dockerfile instructions](#dockerfile-instructions)
- [`RUN` instruction](#run-instruction)
- [Build image from `Dockerfile`](#build-image-from-dockerfile)
- [`CMD` instruction](#cmd-instruction)
- [`ENTRYPOINT` instruction](#entrypoint-instruction)
- [Start and stop a contanier](#start-and-stop-a-contanier)
- [Terminal access to a container](#terminal-access-to-a-container)
- [Deleting container](#deleting-container)
- [Deleting image](#deleting-image)
- [Pushing images to Docker Hub](#pushing-images-to-docker-hub)
- [Volumes](#volumes)
  - [Mount a Volume](#mount-a-volume)
  - [`VOLUME` in `Dockerfile`](#volume-in-dockerfile)
  - [Lets run a `tomcat` container with `volume` `/www`](#lets-run-a-tomcat-container-with-volume-www)
- [Network basics](#network-basics)
  - [`port` mapping](#port-mapping)
  - [`EXPOSE` instruction](#expose-instruction)
  - [Linking containers](#linking-containers)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

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

## `docker commit`

- `docker commit` saves changes in a `container` as a new `image`
- `$ docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`
  - if no [TAG] is specified, `docker` will use `latest`
  - if no `REPOSITORY` is specified `REPOSITORY` and `TAG` will be `none`
- let's see it in action
  - `$ docker run -it ubuntu bash`
  - in the `ubuntu` container `$ apt-get install -y curl`
    - if you find `E: Unable to locate package curl` error
      - `$ apt-get -qq update`
      - `$ apt-get -qq -y install curl`
      - refer [this StackoOverflow post](http://stackoverflow.com/questions/27273412/cannot-install-packages-inside-docker-ubuntu-image) for further details
  - if the `prompt` seems to be hanging, try pressing `return` or `ctrl + c` and retry
  - `exit` from the container
  - list the container
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
  - run the image, with `docker run`
  ```sh
  $ docker run -it curl bash
  root@86d61ffb3dac:/# which curl
  /usr/bin/curl
  root@86d61ffb3dac:/#
  ```

## Dockerfile

### What is Dockerfile?

A `Dockerfile` is a configuration file that contains instructions for building a `Docker image`
- more effective than docker commit
- create image multiple times via `docker build` without carrying out the redundant steps manually
- easily fits in CI/CD process

### Dockerfile instructions

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

- each `RUN` instruction will execute the command on top of `writable layer` and `commit` the changes
- aggregate multiple `RUN` instructions using `&&` to avoid multiple `commit`
```Dockerfile
RUN apt-get -qq update && apt-get -qq -y install curl
```

## Build image from `Dockerfile`

- `$ docker build [OPTIONS] PATH | URL | -`
- `PATH` is the build `context`
- `docker build` will look for `Dockerfile` at the root of the `context`
  - use `-f <file-name>` to specify a custom docker file
- option `-t [repository:tag]` will tag the `image`
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
- verify the image built via `docker images`
```sh
$ docker images
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
curl                      latest              33374acd807e        2 minutes ago       173.5 MB
```

## `CMD` instruction

- `CMD` defines the default command that will be executed when the container is executed
- `CMD` no role during `image` creation
- **only one** `CMD`, the last one wins
- can be overridden at run time
- `CMD` can be in  three forms
  - `CMD ["executable","param1","param2"]` (exec form, this is the preferred form)
  - `CMD ["param1","param2"]` (as default parameters to ENTRYPOINT)
  - `CMD command param1 param2` (shell form)
- create a new image with `CMD`
```Dockerfile
FROM ubuntu

RUN apt-get -qq update
RUN apt-get -qq -y install curl
RUN apt-get -qq -y install iputils-ping

CMD ["ping", "127.0.0.1",  "-c", "30"]
```
- `$ docker build -t ping .` to build a new image
- run the new image via `docker run`
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

An `ENTRYPOINT` allows you to configure a container that will run as an executable.
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

- `$ docker build -t ping .` to build a new image
- run the new image via `docker run` without any arguments

```sh
$ docker run ping
Usage: ping [-aAbBdDfhLnOqrRUvV] [-c count] [-i interval] [-I interface]
            [-m mark] [-M pmtudisc_option] [-l preload] [-p pattern] [-Q tos]
            [-s packetsize] [-S sndbuf] [-t ttl] [-T timestamp_option]
            [-w deadline] [-W timeout] [hop1 ...] destination
```

- run the new image via `docker run` with arguments

```sh
$ docker run ping 127.0.0.1 -c 5
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.043 ms
...
5 packets transmitted, 5 received, 0% packet loss, time 3996ms
rtt min/avg/max/mdev = 0.037/0.040/0.043/0.008 ms
```

## Start and stop a contanier

- `docker start <container-id>` to start a container
- `docker stop <container-id>` to stop a container
- `docker ps` to list all the active containers and their corresponding ids
- `docker ps -a` will list all the containers including exited ones

## Terminal access to a container

- `$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...]` to start the command in the given container
- run a long running process in a docker container in detached mode
```sh
$ docker run -d tomcat
e00264199cf66426aafcf5df48c5ee2b2ddf408bee62d357c2d1d23791f58c26
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e00264199cf6        tomcat              "catalina.sh run"   6 seconds ago       Up 5 seconds        8080/tcp            pensive_engelbart
```
- run `bash` in the already running container
```sh
$ docker exec -i -t e00264199cf6 bash
root@e00264199cf6:/usr/local/tomcat#
```
- exiting from this process will not stop the container as this is not the process with `pid 1`
```sh
root@e00264199cf6:/usr/local/tomcat# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  1 17:41 ?        00:00:03 /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/loggin
root        32     0  0 17:43 ?        00:00:00 bash
root        39    32  0 17:44 ?        00:00:00 ps -ef
```

## Deleting container

- only the stopped containers can be deleted
- `$ docker rm contanier-id`
- `$ docker rm $(docker ps -aq)` will delete all the stopped container

## Deleting image

- `$ docker rmi <image>`
  - <image> could be either image-id or repo:tag

## Pushing images to Docker Hub

- create an account, if you dont have one at [hub.docker.com](https://hub.docker.com)
- navigate to [add repo](https://hub.docker.com/add/repository/)
  - and follow the instructions
- once the repo is created at [hub.docker.com](https://hub.docker.com), use `$ docker push repo:tag` to push it to docker hub.
- prior pushing authenticate docker via hub.docker.com credentials
  - `$ docker login` and follow the promt and instrunctions
- `$ docker push sarbbottam/hello-world`

## Volumes

A **Volume** is a designated directory in a container, which is designed to persists data, independent of the container's life cycle

- volume changes are excluded when updating an `image`
- persists when a `container` is deleted
- can be mapped to a `host` folder
- can be shared between `containers`

### Mount a Volume

- Volumes are mounted when creating or executing a container
- Volume paths must be absolute

### `VOLUME` in `Dockerfile`

- cannot map to host directories
  -  use `docker run -v` to map to a host directory
```Dockerfile
VOLUME /www
```

### Lets run a `tomcat` container with `volume` `/www`

```sh
$ docker run -d -P -v /www tomcat
b65bcc2e397d9553e7cbea34a9e8c557d988469264406d2510dcef687aae3812
```

Let's verify the `volume` `/www`
```sh
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
b65bcc2e397d        tomcat              "catalina.sh run"   16 seconds ago      Up 15 seconds       0.0.0.0:32768->8080/tcp   compassionate_morse

$ docker exec -it b65bcc2e397d bash
root@b65bcc2e397d:/usr/local/tomcat# cd /
root@b65bcc2e397d:/# ls
bin   dev  home  lib64	mnt  proc  run	 srv  tmp  var
boot  etc  lib	 media	opt  root  sbin  sys  usr  www
root@b65bcc2e397d:/#
```

## Network basics

### `port` mapping
- use `-p host-port:container-port`
- use `-P` for auto-mapping
  - `docker` will auto map the contanier-port to container-port from `49153` to `65535`
  - **only works for the ports sepecified via `EXPOSE` instruction in `Dockerfile`

### `EXPOSE` instruction

configures whihc ports a container will listen on at runtime

```Dockerfile
EXPOSE 80 443
```

### Linking containers

Linking is a communication method between containers which allows them to securely transfer data from one to another

- recipient containers have access to data on source containers
- links are established based on container names
- create a named source container
- create recipient container and use `--link` to link

```sh
$ docker run -d --name database postgres
$ docker run -d -P --name website --link database:db nginx
```

```sh
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                           NAMES
d4625c4ff573        nginx               "nginx -g 'daemon off"   27 minutes ago      Up 27 minutes       0.0.0.0:32770->80/tcp, 0.0.0.0:32769->443/tcp   website
32d6751b1d88        postgres            "/docker-entrypoint.s"   28 minutes ago      Up 28 minutes       5432/tcp                                        database
```

```sh
$ docker exec -it d4625c4ff573 bash
root@d4625c4ff573:/# cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	db 32d6751b1d88 database
172.17.0.4	d4625c4ff573
```

```
root@d4625c4ff573:/# exit
$ docker inspect 32d6751b1d88 | grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.3",
                    "IPAddress": "172.17.0.3",
```

- useful for micro service architecture
  - tomcat hosting the web-application in a separate container
  - mysql is running in a separate container
