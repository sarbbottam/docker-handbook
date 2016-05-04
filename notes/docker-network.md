🐳 Docker Network Basics
---

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [`port` mapping](#port-mapping)
- [`EXPOSE` instruction](#expose-instruction)
- [Linking containers](#linking-containers)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

## `port` mapping
- use `-p host-port:container-port`
- use `-P` for auto-mapping
  - `docker` will auto map the contanier-port to container-port from `49153` to `65535`
  - **only works for the ports** sepecified via `EXPOSE` instruction in `Dockerfile`

## `EXPOSE` instruction

configures to which `ports`, a `container` will listen on at runtime

```Dockerfile
EXPOSE 80 443
```

## Linking containers

Linking is a communication method between `containers`, which allows them to securely transfer data from one to another

- recipient `containers` have access to data on source `containers`
- links are established based on container names
- useful for micro service architecture
  - tomcat hosting the web-application in a separate container
  - mysql is running in a separate container

Let's see it in action
 - create a named `source container`
 - create `recipient container` and use `--link` to link

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