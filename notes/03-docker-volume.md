🐳 Docker Volume
---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Volumes](#volumes)
- [Mount a Volume](#mount-a-volume)
- [`VOLUME` in `Dockerfile`](#volume-in-dockerfile)
- [Lets run a `tomcat` container with `volume` `/www`](#lets-run-a-tomcat-container-with-volume-www)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

## Volumes

A **Volume** is a designated directory in a container, which is designed to persists data, independent of the container's life cycle

- volume changes are excluded when updating an `image`
- persists when a `container` is deleted
- can be mapped to a `host` folder
- can be shared between `containers`

## Mount a Volume

- Volumes are mounted when creating or executing a container
- Volume **paths must be absolute**

## `VOLUME` in `Dockerfile`

- **cannot map to host directories**
  -  use `docker run -v` to map to a host directory
```Dockerfile
VOLUME /www
```

## Lets run a `tomcat` container with `volume` `/www`

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
